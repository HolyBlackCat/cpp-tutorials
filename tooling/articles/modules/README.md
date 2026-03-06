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

This means that we no longer need to emit the header dependencies as the byproduct of the compilation (unlike pre-modules), since it can be done during scanning, and is needed to correctly rescan anyway (in theory, the alternative to emitting them during scans is to emit them during compilation, but then if the subsequent compilation of this TU fails, you would have to remember to rescan it; this seems unnecessarily complicated).

But if rebuilding a BMI produced a bitwise identical file (a build system should compare hashes), then TUs importing it don't have to be rebuilt. This can happen e.g. if the imported module unit defines a function, and only its body has changed, assuming the compiler is sufficiently clever to not have the body affect the BMI.

## Scanning dependencies

The scan commands need the same flags as your compiler: include paths, macros, language standard, etc.

The compilers have converged on the common JSON-based format for outputting module scan results, called [`P1689R5`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1689r5.html) after the proposal that introduced it.

The format itself seems to allow for scanning multiple source files by a single command (see the [`"rules": [...]` array](#p1689r5-schema-summary)), but none of the three compilers I tested can actually do this, from what I understand. The only compiler that attempts this is MSVC, but it outputs multiple braced groups in a row (`{...}{...}`) instead of filling the `"rules"` array, resulting in malformed JSON.

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

There is also `-fprebuilt-module-path=...` to search BMIs in a directory, but then they need to be named in a particular way (matching the module name), and unlike other compilers, Clang never selects the BMI names automatically, so we'd have to ensure the right names ourselves. For this reason, I would prefer `-fmodule-path=...` (and because this is the only thing MSVC supports, so you need this logic anyway in your build system).

### GCC

GCC treats and `.cpp` and `.cppm` the same way.

GCC needs `-fmodules` to export or import modules.

GCC somehow allows modules in any language version, but passing `-std=c++20` just in case is still a good idea.

You don't need any special flags when compiling a BMI, the following command just works.
```sh
g++ a.cppm -std=c++20 -fmodules -c a.o
```
GCC automatically decides where to place the BMIs and where to import them from, it puts them in the `./gcm.cache` directory. So no special flags are needed when importing the modules.

To override where the BMIs are stored and loaded from, you can provide a **module map** file, which is just a list of all module units, one per line: name and BMI path, separated by a space, e.g.:
```
A blah/a.gcm
A:P bleh/a_p.gcm
```
And so on. You can also add `$root foo/bar` as the first line, to prepend that directory (`foo/bar`) to every other path in the file.

This file can then be passed to `-fmodule-mapper=...` to any GCC command that may export or import a module. If this file is specified, then it must list every module, since it disables the automatic name-to-path translation.

GCC can also [interact with programs over sockets or otherwise](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html) instead of using a simple file, but this sounds like too much work for me.

### MSVC

MSVC doesn't care about file extensions for importable vs non-importable units.

But MSVC doesn't understand the `.cppm` extension, so if you use that, you need `/TP` to force treat it as C++.

The standard needs to be set to `/std:c++20` or newer.

MSVC needs different flags for different kinds of importable units: interface units need `/interface`, and internal partitions need `/internalPartition`.

Like Clang, MSVC needs `/reference NAME=PATH` for every imported module recursively. It seems that unlike with Clang, we can skip it for indirect non-exported dependencies (so e.g. `export module A;` does `import B;`, and then a TU does `import A;`, then that TU doesn't need `/reference B=...` because `B` is not `export import`ed). But since the scans don't provide this information (whether something is `import`ed or `export import`ed), this is not useful and we have to recurse into all dependencies anyway.

MSVC *also* searches for BMIs in the current directory, and it seems this search can't be disabled, and custom directories can't be added.

### Compiler flags summary

This table is simplied to only explain what I consider the recommended build procedure. More details are explained per compiler above.

Category|Clang|GCC|MSVC|Comment
---|---|---|---|---
Language standard|`-std=c++20` or newer|Any standard + `-fmodules`|`/std:c++20` or newer
Other flags|`-fmodules-reduced-bmi` to use a nicer BMI format (default since Clang 22)|No|`/nologo /EHsc` recommended out of general sanity
Need a module map?|No|Yes, [`-fmodule-mapper=...`](#gcc-1)|No|The module map can be skipped if you don't mind the BMIs being in `./gcm.cache`. I think custom locations should be preferred.
Need to list imported BMI paths? |`-fmodule-path=NAME=PATH`|No|`/reference NAME=PATH`|Must list indirect dependencies too.
Flags for different kinds of module units|`-xc++-module` for importable units (optional for `.cppm`)<br/>`-xc++` otherwise (optional for `.cpp`)|No|`/interface` for interface units<br/>`/internalPartition` for implementation partitions

## Clang's two-stage compilation

As an alternative to the single-stage compilation described above (that produces both the BMI and the `.o` in a single compiler invocation), Clang has several two-stage compilation models to choose from, where the compiler is called twice per importable module unit: first to produce the BMI, and then to produce the object file.

Here they are:

```
1.
    a.cppm  --1-->  full BMI  --2-->  a.o
                      |
                      '-->  consumers

2.
                 .-->  full BMI  --2-->  a.o
    a.cppm  --1--|
                 '-->  reduced BMI  -->  consumers

3.
       .-----1-->  a.o
    a.cppm
       '-----2-->  BMI  -->  consumers
```

Clang has "full BMIs" vs "reduced BMIs". A full BMI is needed to be able to produce an `.o` from it, but otherwise a reduced BMI is better.

Single-stage compilation (and `3` here) should use reduced BMIs, as those seem to be more optimal, but full BMIs should work for those too.

Reduced BMIs seem to be required if you want to avoid rebuilding importers of a module when its source file changes, but the interface doesn't change (since full BMIs have to include function bodies and such, making the file hash different when a function body changes).

`-fmodule-file=...` must be passed to **both** stages, even in approach `2`.

* **`1-2` and `2-2`**: To produce an `.o` from a full BMI, just pass it instead of the source file, with `-c`. If the extension of the BMI is not `.pcm`, use `-xpcm` before the file.

* **`1-1`**: To produce a full BMI instead of an `.o`, pass `--precompile` instead of `-c`. (This sets the default to `-fno-modules-reduced-bmi` even on newer Clang versions.) `-o` sets the BMI path then.

* **`2-1`**: To produce both a full BMI and a reduced BMI, use `--precompile -fmodules-reduced-bmi -fmodule-output=a_reduced.pcm -o a_full.pcm`. Forgetting `-fmodules-reduced-bmi` on Clang 22 or newer does nothing, but on Clang 21 it silently doesn't emit the reduced BMI.

* **`3-1`**: Just do `-xc++` to produce an `.o` without a BMI.

* **`3-2`**: To produce a reduced BMI without also producing a full BMI or `.o`, use:

  * `--precompile-reduced-bmi` instead of `--precompile` — Needs Clang 23 or newer (which wasn't yet released at the time of writing this).

  * This could be imitated using `2-1`, sending the full BMI to `/dev/null`, but that seems slow enough to make approach 3 useless in Clang 22 and older.

It's unclear which of those strategies is the best, and if they are better than the single-stage approach. Benchmarks are needed.

Something tells me `1` is not a good strategy, and good strategy would involve reduced BMIs (so either `2`, `3`, or single-stage).
