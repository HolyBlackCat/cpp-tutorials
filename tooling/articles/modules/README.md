# C++20 modules for build system enthusiasts

I was curious about C++20 modules, but I don't just want to flip a magical switch in CMake and have them work. I want to actually understand how they work on the build system level, to be able to build them manually with a makefile if I want to, and so on.

So I did some research and documented the results here.

## What are modules

The TL;DR is that modules are a new mechanism that was added to C++20 that can replace headers, bringing faster compilation times and isolating components from each other (no cross-talk via macros like we had between headers).

Large C++ projects are often split into several source directories, similar to this one:

```cpp
A/
├── A.h
│       #pragma once
│       #include "foo.h"
│       #include "bar.h"
│
├── foo.h
│       #pragma once
│       void foo();
├── foo.cpp
│       #include "foo.h"
│       void foo() {}
│
├── bar.h
│       #pragma once
│       void bar();
├── bar.cpp
│       #include "bar.h"
│       #include "internal.h"
│       void bar() {internal();}
│
├── internal.h
│       #pragma once
│       void internal();
├── internal.cpp
│       #include "internal.h"
│       void internal() {}
│
├── internal2.h
│       #pragma once
│       void internal2();
└── internal2.cpp
        #include "internal2.h"
        void internal2() {}
```

Here:

* `A.h` is the main header that includes all other **public** headers in this directory. (So `foo.h` and `bar.h` in this example.)

   Those are done for convenience, not everyone has those.

* `foo.h|.cpp`, `bar.h|.cpp`, `internal.h|.cpp`, and `internal2.h|.cpp` are the usual header+source pairs, except that `internal.h` and `internal2.h` are considered to be an internal header that the users shouldn't include directly (those are often placed in a `detail` subdirectory).

An entire such directory maps to a single C++20 module (more correctly called a **"named module"** — here and below bold `"..."` indicate official terms from the standard).

The separation between the files doesn't change. You get the following file structure:

```cpp
A/
├── A.cppm
│       export module A;
│       export import :foo;
│       export import :bar;
│
├── foo.cppm
│       export module A:foo;
│       export void foo();
├── foo.cpp
│       module A;
│       void foo() {}
│
├── bar.cppm
│       export module A:bar;
│       export void bar();
├── bar.cpp
│       module A;
│       import :internal;
│       void bar() {internal();}
│
├── internal.cppm
│       module A:internal;
│       void internal();
├── internal.cpp
│       module A;
│       import :internal;
│       void internal() {}
│
├── internal2.cppm
│       module A:internal2;
│       void internal2();
└── internal2.cpp
        module A;
        import :internal2;
        void internal2() {}
```

Here `A` is the module name. It's either a single identifier or a `.`-separated list of identifiers, so e.g.: `A` or `A.B` or `A.B.C`. The `.` doesn't have a special meaning, it's just a part of the name. It doesn't have to match the directory name, but it probably should (prefix it with your project's name).

The first line of each of those files (either `module ...` or `export module ...`) is called the **"module declaration"**.

All of the files with module declarations in them are collectively called the **"module units"** (since each of them needs to be compiled, unlike headers, each is also a TU — a translation unit).

The `.cppm` files with `export module` in them are the **"module interface units"**. Again, unlike headers, those need to be compiled.

The one `.cppm` file with `export module A;` is the **"primary module interface unit"**. This is the only required part of a module. This is the only part that's importable from outside of the module. It's supposed to `export import` all other interface units in a module (possibly indirectly, see below) (the program is IFNDR otherwise, but the worst this could do is cause build system issues, if your build system can't handle this). While not shown here, the primary module interface unit can also declare things directly, instead of just importing other files.

The non-primary interface units are informally called the **partition interface units**. The part after `:` is the **"partition name"**, and has the same rules as module names (a `.`-separated list of identifiers). The partition names are internal to the module, and need to be unique only within this module. The partition name should probably match the filename, but it doesn't have to. Partition interface units can `export import` each other, as long as there's no circular dependencies between them (so the primary interface unit doesn't have to reexport all partitions directly, it can also do it indirectly through interface partitions).

The `.cpp` files with `module A;` and `module A:blah;` in them are the **"module implementation units"**: the regular ones and (informally) the **partition implementation units** (or **internal partition units**) respectively.

The regular implementation units implicitly `import A;` (import the entire module), while the partition implementation units don't.

The partition implementation units are basically a replacement for internal headers, as their contents can't be used outside of the module (unlike interface partitions, they can't be `export import`ed, only `import`ed).

So you have 4 new kinds of files now: (4 kinds of module units)

&nbsp;|<code><b>export</b> module ...</code><br/>("module interface unit")|<code>module ...</code><br/>("module implementation unit")
---|---|---
`[export] module A;`|"primary module interface unit"<br/>Must reexport all non-primary interface units.|Most implementations go here.<br/>Implicitly does `import A;`.
<code>\[export\] module A<b>:blah</b>;</code><br/>("module partition unit")|Must be reexported by the primary module interface unit.<br/>Can be imported and export-imported only by other TUs in the same named module.|Can be imported only by other TUs in the same named module.

The 3 kinds other than the non-partition implementation units are called **importable module units**, you can `import` those (partitions — only in the same named module).

I'm using [the file extensions recommended by Clang](https://clang.llvm.org/docs/StandardCPlusPlusModules.html#file-name-requirements), which are `.cppm` for importable units and `.cpp` for non-importable ones (for non-partition implementation units), but this isn't mandatory, and I'm not sure if the C++ world has settled on a single convention yet.

The separation between declarations and definitions works the same way with modules as it did with headers and `.cpp` files: you should do it if you value your incremental build times, but if you want your small functions to be inline-able, they should be defined in the interface units.

Compilers and build systems **can** be clever and improve incremental build speed by not rebuilding the TUs that import the module units that haven't changed, but this needs both compiler support and build system support. Until you make sure your build system and all compilers you're using can do this, you should keep separating declarations into the interface units.

## The build procedure

All module TUs are compiled, even the interface units (unlike headers, which are not compiled individually, only as a part of `.cpp` files they're included into).

In addition to an `.o` file, you also need to produce a BMI (built module interface, this is how Clang calls it; GCC calls it CMI, a compiled module interface; MSVC calls it IFC, which apparently stands for "interface").

You need the BMI file to `import` a module unit, so those are only needed for importable units (i.e. excluding non-partition implementation units).

The `.o` file can often end up basically empty, if the module unit just acts as a simple header and doesn't have any definitions in it; then not linking it doesn't cause issues (but it's easier to link them unconditionally).

The BMIs need to be produced in a particular order: imported BMIs need to be produced before the BMIs of units importing them (and before compiling the units importing them).

`.o` and BMI can be produced separately. There are three approaches to compiling modules:

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

## Compiler difference summary

&nbsp;|GCC|Clang|MSVC|Comment
---|---|---|---|---
Importable module extension|Any C++ extension works, including `.cpp` and `.cppm`.|`.cppm` (maybe others), otherwise must build with `-xc++-module`.<br/>`.cpp` doesn't work without `-xc++-module`.|Any C++ extension works, but `.cppm` doesn't work (unless forced with `/TP`)|I like the Clang's convention. It works on GCC and Clang, but needs `/TP` on MSVC.
Chooses the BMI name?|✅From module name.|❌From input filename.|✅From module name.|On Clang, if you want it to automatically search directories for modules, you must manually ensure the correct BMI naming.
Searches for modules where?|In directory `./gcm.cache/` by default.<br/>Can override locations for all modules in a file passed to `-fmodule-mapper=...`.<br/>Can override just the dir with <code>-fmodule-mapper="\|@g++-mapper-server --root Dir"</code> if your distro ships `g++-mapper-server`. Or you can implement a [custom mapper server](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Module-Mapper.html).|Nowhere by default.<br/>Can pass individual modules to `-fmodule-file=NAME=PATH`.<br/>Can pass directories to `-fprebuilt-module-path=DIR` (but this requires manually selecting the right BMI filename when producing it).|Current directory by default (this can't be disabled).<br/>Can pass individual modules to `/referenceNAME=PATH`.|Recommendation: Pass individual modules on Clang and MSVC, and produce a module map file for GCC.
Proper two-phase compilation|❌|✅|❌


## How the build systems scan the source files

To ensure the correct build order, a build system naturally needs to know, for each `.cpp`/`.cppm` file:

1. Whether it's a module unit, and if so, what kind, and its module name and partition name if any. We then need to create a reverse mapping from module name + partition name back to the filename.

2. What module units does it import (directly, not recursively).

This is in addition to the pre-modules requirement of knowing the list of headers that a file includes recrusively, but as usual this is only needed *after* the initial compilation, and is a byproduct of the compilation, unlike (1) and (2), which need to be collected for all files before the compilation.

(1) and (2) need to be determined before any compilation and BMI generation can take place.

### Scanning for module information

The standard format for this information is JSON-based, it's called [`P1689R5`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p1689r5.html), named after the proposal that introduced it.

Both GCC, Clang, and MSVC support this format, but some of them also have other non-standard formats.

Notably this format doesn't contain a list of included headers. You need to obtain this as usual when compiling, in compiler-specific formats.

Notice that this scan has to be performed before any BMIs are built, so it doesn't consume any BMIs.

#### GCC, non-standard

```sh
g++ a.cppm -std=c++20 -M -fmodules
```

This lists both the provided and imported modules, and also the included headers, in the classical Makefile-like format (that's used by GCC and Clang to report included headers).

It seems the GCC people have came up with a way to amend this format to include module information.

Since it's GCC-specific, I won't dig too deep into it.

#### MSVC, non-standard

MSVC uses `/c /sourceDependencies...` for this, where `...` is either a filename or `-` to print the information to stdout.

This outputs both the provided and imported modules, and also included headers, in a non-standard MSVC-specific JSON format.

Interestingly, this requires passing `/interface` or `/internalPartition` to interface untis and internal partition units respectively, but I'd hope to obtain this information for this very flag, which makes it useless for handling modules.

#### GCC, standard P1689R5

```sh
g++ a.cppm -std=c++20 -M -fmodules -fdeps-format=p1689r5 -fdeps-file=a.json
```
This writes the description to `a.json`.

You can also add `-fdeps-target=a.o` to adjust the output filename reported in the result, but said filename is not very useful.

This sadly prints unwanted information to stdout, so it has to be silenced. The compiler errors if any go to stderr, so we don't lose anything. The stdout output can be redirected with `-o`, but since it's not useful, `>/dev/null` seems more promising.

#### MSVC, standard P1689R5
```sh
cl a.cppm /EHsc /std:c++latest /scanDependenciesa.json /TP
```
This writes the description to `a.json`.

MSVC seems to have no equivalent to `-fdeps-target=...`, so the output filename reported in the JSON can't be adjusted, but again it doesn't seem to be very useful.

#### Clang, standard P1689R5

Clang ships a separate utility called `clang-scan-deps` for this:
```sh
clang-scan-deps -format=p1689 -o a.json -- clang++ a.cppm -std=c++20
```
As is usual with Clang-based utilities, the part after `--` is the simulated compiler command.

This writes the JSON to `a.json`. Omitting `-o ...` writes it to stdout instead.

Adding `-o ...` after `--` changes the output filename reported in the JSON, but again that doesn't seem to be very useful.

Omitting `-format=p1689` outputs the included headers instead, in Makefile-like style. Unlike GCC, this doesn't add module information.

### Example scan results

Let's look at the scan results of this simple module unit:
```cpp
export module A;
import B1;
import B2;
import :P1;
import :P2;
```

We get the following json:

```json
{
  "revision": 0,
  "rules": [
    {
      "primary-output": "a.out",
      "provides": [
        {
          "is-interface": true,
          "logical-name": "A",
          "source-path": "a.cppm"
        }
      ],
      "requires": [
        {
          "logical-name": "B1"
        },
        {
          "logical-name": "B2"
        },
        {
          "logical-name": "A:P1"
        },
        {
          "logical-name": "A:P2"
        }
      ]
    }
  ],
  "version": 1
}
```

Then with `export module A:P;`:

```json
{
  "revision": 0,
  "rules": [
    {
      "primary-output": "a.out",
      "provides": [
        {
          "is-interface": true,
          "logical-name": "A:P",
          "source-path": "a.cppm"
        }
      ],
      "requires": [
        {
          "logical-name": "B1"
        },
        {
          "logical-name": "B2"
        },
        {
          "logical-name": "A:P1"
        },
        {
          "logical-name": "A:P2"
        }
      ]
    }
  ],
  "version": 1
}
```

And with `module A:P;`:

```json
{
  "revision": 0,
  "rules": [
    {
      "primary-output": "a.out",
      "provides": [
        {
          "is-interface": false,
          "logical-name": "A:P",
          "source-path": "a.cppm"
        }
      ],
      "requires": [
        {
          "logical-name": "B1"
        },
        {
          "logical-name": "B2"
        },
        {
          "logical-name": "A:P1"
        },
        {
          "logical-name": "A:P2"
        }
      ]
    }
  ],
  "version": 1
}
```

And with `module A;`. Notice the lack of `"provides"`, and the implicit import of `A`.

```json
{
  "revision": 0,
  "rules": [
    {
      "primary-output": "a.out",
      "requires": [
        {
          "logical-name": "B1"
        },
        {
          "logical-name": "B2"
        },
        {
          "logical-name": "A:P1"
        },
        {
          "logical-name": "A:P2"
        },
        {
          "logical-name": "A"
        }
      ]
    }
  ],
  "version": 1
}
```


And lastly a non-module unit:
```cpp
import B1;
import B2;
```

```json
{
  "revision": 0,
  "rules": [
    {
      "primary-output": "a.out",
      "requires": [
        {
          "logical-name": "B1"
        },
        {
          "logical-name": "B2"
        }
      ]
    }
  ],
  "version": 1
}
```

### Analyzing scan results

What can we see from those results?

1. Importable module units are indicated by the presence of `rules.provides: [...]`.

   * Out of those, interface units are indicated by `rules.provides.is-interface: true/false`.

2. Partitions are handled for us, they don't need to be special-cased. Similarly, the automatic `import` in the non-partition implementation unit is automatic.

3. Non-partition implementation units are indistinguishable from non-module TUs.
