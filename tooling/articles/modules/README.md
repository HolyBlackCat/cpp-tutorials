# How C++20 modules are compiled

## Pre-C++20 compilation model

Each `.cpp` file is compiled independently. As a byproduct of the compilation, you get a list of headers that were included directly or indirectly.

Then on a rebuild, you check the modification times of the headers from that list, and if any of the headers were modified, you recompile the respective the `.cpp` file.

## What are C++20 modules

Modules are an alternative to headers, but they need to be precompiled (into a compiler-specific format) before being imported, and importing those precompiled files is way faster than including headers.

Different compilers have different conventions for what to call those precompiled module files. I'm going to call them BMI, following Clang's convention.

Compiler|Name|Default extension
---|---|---
Clang|BMI (built module interface)|`.gcm` (GNU compiled module?)
GCC|CMI (compiled module interface)|`.pcm` (precompiled module)
MSVC|IFC (stands for "interface"?)|`.icf` (stands for "interface"?)

Unlike headers, modules don't leak macros: importing a module doesn't give you its macros, and importing a module isn't affected by the macros you have defined.

In addition to a BMI, compiling a module also produces an object file (`.o`), which should be linked into the final executable. (If the module doesn't contain function definitions and such, then forgetting to link the `.o` doesn't error.)

## A simple example

```cpp
// a.cppm
export module A;

export int sum(int a, int b) {return a + b;}
```
```cpp
// 1.cpp
#include <iostream>
import A;

int main()
{
    std::cout << sum(10, 20) << '\n';
}
```

Here are simple instructions how to compile this, a more detailed explanation is provided later.

* Clang

  ```sh
  clang++ a.cppm -std=c++20 -c
  clang++ 1.cpp -std=c++20 -fmodule-file=A=a.pcm -c
  clang++ a.o 1.o
  ```

* GCC

  ```sh
  g++ a.cppm -fmodules -std=c++20 -c
  g++ 1.cpp -fmodules -std=c++20 -c
  g++ a.o 1.o
  ```

* MSVC

  ```sh
  cl /nologo /EHsc a.cppm /TP /std:c++20 /interface /c
  cl /nologo /EHsc 1.cpp /std:c++20 /c
  cl a.obj 1.obj
  ```

In each of those, the first command both produces a BMI and an object file for the module. The second command consumes (imports) that BMI and produces an object file for the file consuming the module. Then the third command links the two object files together. (Notice that with Clang, we need to tell it where to find the BMI for a specific module. More on the differences between compilers later.)

Since a BMI is faster to produce than an object file, compilers can employ various tricks to inform the build system the BMI is ready, so that it can be consumed (by a parallel compilation command) without waiting for the object. More on that later too.

There seems to be no single convention for what the file extension should be. I'm using the Clang's convention of `.cppm`. Since MSVC doesn't understand this extension, I'm using `/TP` to tell it that it's a C++ source file.

`A` is the module name. Module names are `.`-separated lists of identifiers, so `A.B` or `A.B.C` would also be valid names. `.` has no special meaning and is just a part of the name.

Notice that the **"module declaration"** (here and below bold quotes indicate the terms from the C++ standard) — the `export module A;` in this example — must be the first non-empty non-comment line in its source file. The module declaration is considered a preprocessor directive, despite not starting with `#` (but `import ...` isn't one).

Source files (translation units) containing a module declaration are called the **"module units"**.

If you need to include any headers, then you should do following:
```cpp
// a.cppm
module;

#include <iostream>

export module A;

export void SayHello()
{
    std::cout << "Hello!\n";
}
```

First you announce that this is a module using `module;`, include any headers you need, and *then* add a module declaration followed by your own code.

The optional part starting with `module;` and until the module declaration is called the **"global module fragment"**.

The part starting from the module declaration (regardless of whether the global module fragment is present) is called the **"module unit purview"** of this module unit.

All code in global module fragments and in non-module TUs is considered to be belong to a single imaginary global module.

Another option for including headers seems to be to wrap them in `extern "C++" { ... }`. While this works, Clang and MSVC warn about includes after module declarations, and don't silence the warning even in the presence of `extern "C++"`.

## Kinds of module units

There are 4 kinds of module units, having different kinds of module declarations. They can share the same module name, and all module units sharing the name are called a single **"named module"** (or informally just a "module"). From outside of a named module, that named module is only importable in one piece.

1. `export module A;` is the **"primary module interface unit"**. This is the only required module unit in a named module, and there can only be one per named module.

   This is what is imported by `import A;`, and is the only thing that can be imported from outside of this named module.

2. `module A;` is the **"module implementation unit"** (a non-"partition" one, more on those later). You can have **more than one** of those.

   ```cpp
   // a.cppm
   export module A;

   export void foo();
   ```
   ```cpp
   // a_foo.cpp
   module A;

   void foo() {....}
   ```
   You can put implementations into those, like you did with `.h`/`.cpp` prior to modules.

   Those are not importable, neither from outside nor from inside the module.

   `module A;` implicitly does `import A;` (to import the primary interface unit), and it's an error to add your own redundant `import A;`.

   Depending on how clever your build system is, it may or may not avoid rebuilding dependent TUs even if you define functions directly in the interface unit. Moving the definitions to an implementation unit should make this guaranteed, and possibly a bit faster for large module units.

3. `export module A:P;` is a non-primary **"module interface unit"** and a **"partition unit"**, or informally a "partition interface unit". There can be multiple of those.

   The purpose of those is to split large interfaces, to avoid having everything in the primary inteface unit.

   ```cpp
   // a.cppm
   export module A;

   export import :P1;
   export import :P2;
   ```
   ```cpp
   // a_p1.cppm
   export module A:P1;

   export void foo() {....} // Define here or in a separate implementation unit.
   ```
   ```cpp
   // a_p2.cppm
   export module A:P2;

   export void bar() {....} // ^
   ```

   As you can see, those can be imported inside of the same named module. But not from outside (at least not directly, but you can import the primary interface unit from outside that reexports those).

   `P` is a **"partition name"**. Like a module name, it's a `.`-separated list of identifiers, with `.` having no special name and just being a part of the name.

   Partition names must be unique per named module.

   The C++ standard requires that all partition interface units are reexported from the primary interface unit (IFNDR otherwise), but seemingly nothing bad could happen if you forget, it just makes them pointless if you do so. Notice that they can reexport each other, so the primary interface unit doesn't have to do it **directly**, it can also do so indirectly.

4. `module A:P;` is a **"module implementation unit"** like 2, and a **"module partition"** like 3, informally an **internal partition unit**.

   Those are a bit strange, they seem to be intended to be used for internal utilities (internal to this named module), to share code between 2's (between non-partition implementation units).

   ```cpp
   // a.cppm
   module A;

   export void foo();
   ```
   ```cpp
   // a.cpp
   module A;
   import :Helpers;

   void foo() {helper();}
   ```
   ```cpp
   // a_helpers.cppm
   module A:Helpers;

   void helper();
   ```
   ```cpp
   // a_helpers.cpp
   module A;
   import :Helpers; // Not strictly necessary in this example, but necessary in general.

   void helper() {}
   ```

   Those are importable only inside of the same named module, like interface partitions. But unlike interface partitions those can't be exported from the primary interface unit.

   The partition name `P` must be unique per named module across both interface and implementation partitions.

To summarize summary:

&nbsp;|`export` (interface)|no `export` (implementation)
---|---|---
no `:Partition`|(1) primary interface unit|(2) implementation unit
`:Partition`|(3) partition interface unit|(4) internal partition unit

* `1` and `3` are <b>"interface module unit"</b>s.

* `3` and `4` are <b>"partition unit"</b>s.

* `1`, `3`, `4` (but not `2`) are **importable units**, though only `1` can be imported outside of its named module.

  Clang's convention is to use `.cppm` for importable units and `.cpp` for non-importable ones.

## The build procedure for modules

To correctly handle dependencies between modules, all `.cpp`/`.cppm` need to be scanned, to determine what importable units they import, and if they are themselves importable (and if so, under what name).

The scan results for a file need to be updated when the file changes, or when any headers that it includes (maybe indirectly) are modified (because you could wrap an import in an `#ifdef` affected by an include). We **don't** need to rescan when the source files of the module units imported by this file are modified, though.

Then the file needs to be rebuilt if it had to be rescanned, or if any of the source files of the module units it imports were modified, recursively.

But if rebuilding a BMI produced a bitwise identical hash (a build system should compare hashes), then TUs importing it don't have to be rebuilt. This can happen e.g. if the imported module unit defines a function, and only its body has changed, assuming the compiler is sufficiently clever to not have the body affect the BMI.

## Scanning dependencies

### Clang
```sh
clang-scan-deps -format=p1689 -o a.json -- clang++ a.cppm -std=c++20 -M -MP -MF a.d -MQ a.tgt -o a.o
```
Writes P1689R5 module deps to `a.json`, based on `-o ...` before `--`. Omit or `-o -` to print to `stdout`.

Writes header deps to `a.d`, based on `-MF ...`. Change to `-MF -` to print to `stdout` (omitting prints to `a.o` per `-o`).

`-o ...` selects the target filename reported to P1689R5.

`-MQ ...` selects the target filename reported to header deps.


### GCC

```cpp
g++ a.cppm -std=c++20 -M -MP -fmodules -fdeps-format=p1689r5 -fdeps-file=a.json -fdeps-target=a.o -MQ a.tgt -MF a.d
```
Writes P1689R5 module deps to `a.json`, based on `-fdeps-file=...`, change to `-fdeps-file=-` to print to stdout (omitting automatically chooses the filename).

Writes header deps to `a.d`, based on `-MF ...`. Omit or `-MF -` to print to stdout. (`-o` seems to be equivalent to `-MF`).

`-fdeps-target=...` selects the target filename reported to P1689R5.

`-MQ ...` selects the target filename reported to header deps.

Don't strictly need `-std=c++20`, but it's weird to use modules before C++20.

Omitting `-fdeps-format=p1689r5` will output module information in some curious GCC-specific Makefile-style format, to the same file as header deps.


### MSVC

```cpp
cl a.cppm /nologo /EHsc /std:c++latest /scanDependencies a.json /sourceDependencies a.d /TP /Foa.tgt
```

Writes P1689R5 module deps to `a.json`, based on `/scanDependencies ...`. Pass `-` to print to stdout.

Writes header deps to `a.d`, based on `/sourceDependencies ...`. Pass `-` to print to stdout.

`/Fo...` selects the target filename reported to P1689R5.

The header deps are in Microsoft's own JSON format. It doesn't include the target filename, so there's no flag to change it.

Don't strictly need `/nologo /EHsc`.

MSVC doesn't understand the `.cppm` extension by default, using `/TP` to force it to assume the input is C++ code. You can omit this for `.cpp` files if you want.

## P1689R5 schema summary

```jsonc
{
    "version": 1, // 0 on GCC
    "revision": 0,
    "rules": [
        {
            "primary-output": "a.o", // As chosen by a flag, see above.
            "outputs": [...], // MSVC only, not very useful.
            "provides": [ // This module. Only exists if this is an importable module unit.
                {
                    "logical-name": "A", // Name of this module unit.
                    "source-path": "a.cppm", // This source file. GCC doesn't report this.
                    "is-interface": true // True if `export module`, false if `module ...`. The latter only appears for partitions, since for non-partitions the entire `provides` is skipped.
                }
            ],
            "requires": [ // Imported modules. On Clang, this array is omitted if empty.
                {"logical-name": "B1"},
                {"logical-name": "B2"},
                // ...
            ]
        }
    ]
}
```
When partitions are involed, `:...` is just appended to the name string, so they don't need to be speecial-cased.

The implicit `import A;` in files starting with `export A;` is implicitly added to `requires`.

Notice that `"requires": [...]` can't differentiate between `import`s and `export import`s, this doesn't matter for a build system.

## Single-stage compilation

All three compilers can do this, and this is the simplest approach.

"Single stage" refers to the BMI and the `.o` being produced by the same compiler invocation.

I'm told that GCC is able to report in real-time when the BMI is done (it can communicate so [over sockets or otherwise](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html)), but implementing this into a build system sounds like too much effort, so I'm going to ignore this in this tutorial.

### Clang

Clang treats `.cppm` and `.cpp` differently, as importable module units and as non-importable/non-module units respectively. This can be overridden using `-xc++-module` or `-xc++` before the source filename respectively. Trying to compile an importable module unit as `.cpp`/`-xc++` doesn't error, but produces no BMI. Trying to compile a non-importable module unit or a non-module as a `.cppm`/`-xc++-module` errors.

Example compilation command for an importable unit:
```sh
clang++ a.cppm -std=c++20 -fmodules-reduced-bmi -c -o a.o -fmodule-output=a.pcm
```
`-std=c++20` or newer is needed.

`-fmodules-reduced-bmi` enables a more optimal module format, which is the default since Clang 22. This flag is ignored on non-importable module units and on non-modules.

`-fmodule-output=...` controls where the BMI is placed. The default is to use the same location as `-o` with a modified extension.

Any time you import a module unit (no matter if in `.cpp` or in `.cppm`), you must add `-fmodule-path=NAME=PATH` for that module unit, and for everything it imports **recursively**.

### GCC

GCC treats and `.cpp` and `.cppm` the same way.

GCC needs `-fmodules` to export or import modules.

GCC somehow allows modules in any language version, but passing `-std=c++20` just in case is still a good idea.

You don't need any special flags when compiling a BMI, the following command just works.
```sh
g++ a.cppm -std=c++20 -fmodules -c a.o
```
GCC automatically decides where to place the BMIs and where to import them from, it puts them in the `./gcm.cache` directory. So no special flags are needed when importing the modules.

To override where the BMIs are stored, you can provide a **module map** file, which is just a list of all module units, one per line: name and BMI path, separated by a space, e.g.:
```
A blah/a.gcm
A:P bleh/a_p.gcm
```
And so on. You can also add `$root foo/bar` as the first line, to prepend that directory (`foo/bar`) to every other path in the file.

This file can then be passed to `-fmodule-mapper=...` to any GCC command that may export or import a module.

GCC can also [interact with programs over sockets or otherwise](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html) instead of using a simple file, but this sounds like too much work for me.

MSVC
...






---




Approach|GCC|Clang|MSVC|Comment
---|---|---|---|---
1\. Single phase|✅|✅|✅|A single compiler invocation produces both the `.o` and the BMI.
2\. Two-phase parallel|❌*|✅ Since Clang 23.<br/>❌* Before that.|❌|`.o` and the BMI are produced by two separate compiler invocations, which can run in parallel.<br/>* In Clang 22 and older, you can only produce a reduced BMI (which is what you want for this step, see the next row) alongside a full BMI, which you can then throw away.<br/>* In GCC, best you can do is disable the object file generation using `-fmodule-only` (to generate only the BMI first), and then do a second pass to generate BMI+`.o`, and discard that second BMI.
3\. Two-phase sequential|❌*|✅|❌|The first compiler invocation produces a reduced BMI (which is what you `import`) and a full BMI. The full BMI only serves as the input to the second invocation (instead of the source file), which produces the `.o`.<br/>* GCC can apparently report in real time when it's done generating the BMI, so it can be consumed without waiting for the `.o`, but this requires implementing a "module mapper server" in the build system, which adds difficulty.

(1) seems to be the default choice.

(3) may or may not be faster than (1) due to increased parallelism.

It's unclear if (2) makes any sense or not compared to (2), since it causes duplicate work, especially if the compiler doesn't have the proper flags (to separately produce only the BMI and only the `.o`).



## A toy modules example

Let's try building a simple module manually. This is the smallest possible example:

```cpp
// a.cppm
module;
#include <iostream>

export module A;

export void foo() {std::cout << "Hello!\n";}
```

```cpp
// 1.cpp
import A;

int main()
{
    foo();
}
```

Notice that:

* The `a.cppm` starts with a `module;` `#include <...>`. You can do this to include old-style headers in your module. In this case, the first line is always just `module;` to announce that this is a module, then you include things, and immediately after that you do the normal module declaration.

  A more modern way of doing this would be:
  ```cpp
  // a.cppm
  export module A;
  import std;

  export void foo() {std::cout << "Hello!\n";}
  ```
  But this is also more difficult to build, so this is for later.

  The part between `module;` and the module declaration is the **"global module fragment"** (it is optional). The part starting from the module declaration is the **"module unit purview"** of this module unit.

* The module `A` consists of only one file, the primary interface unit. This is the only required part of a module.

* A non-module-unit `1.cpp` is importing a module. This is allowed.

### How to build it

I assume you're already familiar with basics of command-line compilation.

#### GCC (single phase)

```sh
g++ a.cppm -std=c++20 -fmodules -c
g++ 1.cpp -std=c++20 -fmodules -c
g++ a.o 1.o
```
Step 1 produces `a.o` and `gcm.cache/A.gcm` (the BMI).<br/>
Step 2 consumes `gcm.cache/A.gcm`, and produces `1.o`.<br/>
Step 3 is the linking.

You can combine steps 2 and 3 in this simple example: `g++ 1.cpp a.o -std=c++20 -fmodules`.

`-fmodules` is needed to enable modules support. Interestingly, it seems to work even without `-std=c++20`, but it's probably a good idea to explicitly request C++20 or newer.

GCC decides by itself where it wants to write and read the BMIs from, and makes overriding this difficult.

In theory there is `-fmodule-mapper="|@g++-mapper-server --root YourPathHere"` for specifying a custom directory, but it doesn't work on my Arch because they forgot to package `g++-mapper-server`, and it still selects all module names for you. You can also implement a [custom mapper server program](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html) and do whatever you want with it, but that's probably an overkill.

Another (saner) option is providing a file to `-fmodule-mapper=...` that lists the desired BMI location for each module name, one per line (and then pass this file both when creating the BMI and when importing it), e.g. in our case it would contain following:
```
A some/path.gcm
```
Optionally you can add `$root some/other/path` as the first line, this directory will be prepended to everything else in this list.

If this file is specified, then it must list *all* modules. It disables the automatic selection of module file names.

#### MSVC (single phase)

```sh
cl a.cppm /EHsc /std:c++latest /interface /TP /c
cl 1.cpp /EHsc /std:c++latest /c
cl a.obj 1.obj
```

Step 1 produces `a.obj` and `A.icf` (the BMI).

Unlike with GCC, you can choose the BMI location using `/icfOutput...`, where `...` is either a filename or a directory name (then the filename is chosen for you). When passing a directory, end it with a slash (so you don't accidentally create a file with this name if this directory doesn't exist).

Unlike GCC and Clang, MSVC wants to be told if it's an interface unit or an internal partition unit (using `/internalPartition` instead of `/interface`).

MSVC doesn't understand the `.cppm` extension, so we use `/TP` to force treat it as C++.

You must enable C++20 or newer, so one of: `/std:c++20`, `/std:c++23preview`, `/std:c++latest`.

I'm enabling exceptions (`/EHsc`) just for general sanity, and because `<iostream>` uses them.

Step 2 seems to automatically search for modules in the current directory. I didn't find a way to override the directory, but you can override it for individual modules using `/reference...=...`, where the first `...` is the module name and the second `...` is a path to the `.ifc`.

#### Clang (single phase)

```sh
clang++ a.cppm -std=c++20 -fmodules-reduced-bmi -c
clang++ 1.cpp -std=c++20 -fmodule-file=A=a.pcm -c
clang++ a.o 1.o
```

Step 1 produces `a.o` and `a.pcm` (the BMI). Like MSVC and unlike GCC, Clang puts BMI in the current directory by default. You can change the BMI filename with `-fmodule-output=...` (unlike MSVC, this must be a filename and can't be a directory).

`-fmodules-reduced-bmi` is purely an optimization (see the next section for more details), and is the default since Clang 22. You can specify it everywhere, even for non-modules, where it has no effect.

Step 2 consumes `a.pcm`, and produces `1.o`. Unlike GCC and MSVC, there is no implicit module search path. You either must specify where to find each module via `-fmodule-file=Name=Path` (like MSVC's `/reference`), or specify a search directory with `-fprebuilt-module-path=Path` (`Path` = `.` in this example).

Interestingly, unlike with GCC and MSVC, the BMI filename chosen by step 1 seems to be based on the source filename `a.cppm`, and if it doesn't match the module name, then `-fprebuilt-module-path=.` won't be able to find it. It can't find it in this example, unless we rename the BMI to `A.pcm`. For this reason (having to manually choose the correct filenames), `-fprebuilt-module-path=...` doesn't look very useful.

Notice that unlike with GCC, we don't need `-fmodules`. Clang allows this flag, but it seems to have no effect.

For Clang, like for MSVC, `-std=c++20` or newer is required. Unlike GCC, it refuses to handle modules in older standards.

Unlike GCC, Clang is sensitive to the extensions here. If the extension is not `.cppm`, you must use `-xc++-module` before the filename to indicate that it's an importable unit that requires a BMI.

#### Clang (two-phase sequential), with fat BMIs - not recommended

```sh
clang++ a.cppm -std=c++20 --precompile
clang++ a.pcm -std=c++20 -c

clang++ 1.cpp -std=c++20 -fmodule-file=A=a.pcm -c

clang++ a.o 1.o
```

Step 1 produces the BMI only. You can change the output filename with `-o`.<br/>
Step 2 produces the `.o` from the BMI, instead of from the source file.<br/>
Step 3 comsumes the BMI and produces `1.o`.<br/>
And finally, step 4 is the linking.

Here, steps 2 and 3 can run in parallel, since the BMI needed by step 3 (and 2) is produced by step 1.

If you compare the size of the BMI in this compilation model vs the single-phase above, you'll notice that this BMI is about 10 times larger. Those are fat vs reduced BMIs. The fat ones need to store more information, to be able to be compiled into `.o`.

Clang seems to want to transition away from fat BMIs to reduced BMIs. Single-phase defaults to reduced BMIs on Clang 22 or newer, and you can use `-fmodules-reduced-bmi` on older Clang to force reduced BMIs (for single-phase compilation, not for this example).

#### With Clang (two-phase sequential), with reduced BMIs

```sh
clang++ a.cppm -std=c++20 --precompile -fmodules-reduced-bmi -fmodule-output=a.reduced.pcm -o a.full.pcm
clang++ a.full.pcm -std=c++20 -c

clang++ 1.cpp -std=c++20 -fmodule-file=A=a.reduced.pcm -c

clang++ a.o 1.o
```

Here step 1 produces both a full BMI and a reduced BMI.

Then step 2 compiles the full BMI into an `.o`.

Step 3 only needs the reduced BMI.

There's some variance regarding the flags needed for step 1 between Clang versions: Clang 21 needs `-fmodules-reduced-bmi` or it won't generate the reduced BMI at all. `-o` is optional (defaults to `a.pcm` here), but Clang 22 warns if you skip `-o`. So the most sensible option is to specify both `-fmodules-reduced-bmi` and `-o` to be compatible with both versions.

#### With Clang (two-phase parallel) - needs Clang 23+

```sh
clang++ a.cppm -std=c++20 --precompile-reduced-bmi
clang++ -xc++ a.cppm -std=c++20 -c

clang++ 1.cpp -std=c++20 -fmodule-file=A=a.pcm -c

clang++ a.o 1.o
```

I'm unable to test this, as Clang 23 isn't out yet, and I don't want to compile it from source. `--precompile-reduced-bmi` should be added in Clang 23.

In theory, this is same as the previous model, but here `a.o` is built from source, instead of from the full BMI, and step 1 doesn't generate the full BMI at all, only the reduced one.

I'm not sure if this makes sense over the previous model, or if this is slower due to duplicating work.

You can imitate this model on older Clangs by taking step 1 from the previous section, and then ignoring/deleting the resulting full BMI.

## Module compilation on different compilers

&nbsp;|GCC|Clang|MSVC|Comment
---|---|---|---|---
Importable module extension|Any C++ extension works, including `.cpp` and `.cppm`.|`.cppm` (maybe others), otherwise must build with `-xc++-module`.<br/>`.cpp` doesn't work without `-xc++-module`.|Any C++ extension works, but `.cppm` doesn't work (unless forced with `/TP`)|I like the Clang's convention. It works on GCC and Clang, but needs `/TP` on MSVC.
Chooses the BMI name?|✅From module name.|❌From input filename.|✅From module name.|On Clang, if you want it to automatically search directories for modules, you must manually ensure the correct BMI naming.
Searches for modules where?|In directory `./gcm.cache/` by default.<br/>Can override locations for all modules in a file passed to `-fmodule-mapper=...`.<br/>Can override just the dir with <code>-fmodule-mapper="\|@g++-mapper-server --root Dir"</code> if your distro ships `g++-mapper-server`. Or you can implement a [custom mapper server](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html).|Nowhere by default.<br/>Can pass individual modules to `-fmodule-file=NAME=PATH`.<br/>Can pass directories to `-fprebuilt-module-path=DIR` (but this requires manually selecting the right BMI filename when producing it).|Current directory by default (this can't be disabled).<br/>Can pass individual modules to `/referenceNAME=PATH`.|Recommendation: Pass individual modules on Clang and MSVC, and produce a module map file for GCC.
Proper two-phase compilation|❌|✅|❌


### Analyzing scan results

What can we see from those results?

1. Importable module units are indicated by the presence of `rules[0].provides: [...]`.

   * Out of those, interface units are indicated by `rules[0].provides[0].is-interface: true/false`.

2. Partitions are handled for us, they don't need to be special-cased. Similarly, the automatic `import` in the non-partition implementation unit is automatic.

3. Non-partition implementation units are indistinguishable from non-module TUs (apart from the fact that they can't import partitions, but a module unit doesn't necessarily import those).
