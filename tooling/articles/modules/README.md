## What are modules

Large C++ projects often split into several source directories, similar to this one:

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
└── internal.cpp
        #include "internal.h"
        void internal() {}
```

Here:

* `A.h` is the main header that includes all other **public** headers in this directory. (So `foo.h` and `bar.h` in this example.)

   Those are done for convenience, not everyone has those.

* `foo.h|.cpp`, `bar.h|.cpp`, and `internal.h|.cpp` are the usual header+source pairs, except that `internal.h` is considered to be an internal header that the users shouldn't include directly (those often placed in a `detail` subdirectory).

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
└── internal.cpp
        module A;
        import :internal;
        void internal() {}
```

Here `A` is the module name. It's either a single identifier or a `.`-separated list of identifiers, so e.g.: `A` or `A.B` or `A.B.C`. The `.` doesn't have a special meaning, it's just a part of the name. It doesn't have to match the directory name, but it probably should.

The first line of each of those files (either `module ...` or `export module ...`) is called the **"module declaration"**.

All of the files with module declarations in them are collectively called the **"module units"** (since each of them needs to be compiled, unlike headers, each is also a TU — a translation unit).

The `.cppm` files with `export module` in them are the **"module interface units"**. Again, unlike headers, those need to be compiled too.

The one `.cppm` file with `export module A;` is the **"primary module interface unit"**. This is the only required part of a module. It's supposed to `export import` all other interface units in a module (the program is IFNDR otherwise, but the worst this could do is cause build system issues, if your build system is that brittle). While not shown here, the primary module interface unit can also declare things directly, instead of just importing other files.

The non-primary interface units are informally called the **partition interface units**. The part after `:` is the **"partition name"**, and has the same rules as module names (a `.`-separated list of identifiers). The partition names are internal to the module, and need to be unique only within this module. The partition name should probably match the filename, but it doesn't have to.

The `.cpp` files with `module A;` and `module A:blah;` in them are the **"module implementation units"**: the regular ones and (informally) the **partition implementation units** respectively.

The regular implementation units implicitly `import A;` (import the entire module), while the partition implementation units don't.

The partition implementation units are a replacement for internal headers: they can only be imported by other TUs in the same module.

So you have 4 new kinds of files now: (4 kinds of module units)

&nbsp;|<code><b>export</b> module ...</code><br/>("module interface unit")|<code>module ...</code><br/>("module implementation unit")
---|---|---
`[export] module A;`|"primary module interface unit"<br/>Must reexport all non-primary interface units.|Can be imported only by other TUs in the same named module.
<code>\[export\] module A<b>:blah</b>;</code><br/>("module partition unit")|Must be reexported by the primary module interface unit.|Most implementations go here.<br/>Implicitly does `import A;`.

I'm using [file name convention recommended by Clang](https://clang.llvm.org/docs/StandardCPlusPlusModules.html#file-name-requirements), which is to use `.cppm` for interface units and `.cpp` for implementation units, but this but this isn't mandatory, and I'm not sure if the C++ world has settled on a single convention yet.

The separation between declarations and definitions works the same way with modules as it did with headers and `.cpp` files: you should do it if you value your incremental build times, but if you want your small functions to be inline-able, they should be defined in the interface units.

Compilers and build systems **can** be clever and improve incremental build speed by not rebuilding the TUs importing a module unit that hasn't changed, but this needs both compiler support and build system support. Until you're sure your build system and all compilers you're using can do this, you should keep separating declarations into the interface units.

## The build procedure

All module TUs are compiled, even the interface units (unlike headers, which are not compiled individually, only as a part of `.cpp` files they're included into).

In addition to an `.o` file, compiling a module unit also produces a BMI (built module interface) or CMI (compiled module interface) file. BMI and CMI mean exactly the same thing, different people just call them differently.

You need the BMI file to `import` a module unit.

Since non-partition implementation units are not importable, those don't produce BMIs.

And regarding the `.o`, if your module unit just acts as a simple header and doesn't have any definitions in it, then the `.o` file it produces is basically empty, and not linking it doesn't cause issues (but it's easier to just link all of them).

Needing the BMI for `import` dictates the build order: any module unit that gets imported needs to be compiled before other units that import it. In particular, the non-primary interface units must be built before the primary interface unit that imports them

Adding to the confusion, some compilers (at least Clang) can produce `.o` and BMIs with separate compiler invocations, which helps building things in parallel, since the `.o` files are not needed for anything other than the final linking.

When building `.o` and BMI with Clang, you can either build them entirely separately (this produces a "thin BMI"), or you can build a "full BMI" first, and then build `.o` from the full BMI instead of from the source. Thin BMIs seem like a better idea, at least since they offer more opportunities for parallel builds.

## A toy modules example

Let's try building a simple module manually.

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

Notice two things:

* The module `A` starts with a `module;` `#include <...>`. You must do this to include old-style headers in your module. In this case, the first line is always just `module;` to announce that this is a module, then you include stuff, and immediately after that you do the normal module declaration.

  A more modern way of doing this would be:
  ```cpp
  // a.cppm
  export module A;
  import std;

  export void foo() {std::cout << "Hello!\n";}
  ```
  But this is also more difficult to build, so this is for later.

* A non-module-unit `1.cpp` is casually importing a module. This is allowed. What's not allowed is declaring a function in a module but defining it in a non-module, and similarly

## How the build system examines the module files

To ensure the correct build order, a build system naturally needs to know, for each named module:

* Which file is the primary interface unit, and where to find each module partition (both interface partitions and implementation partitions). But it doesn't need much

* GCC:

  * You can get some information in Makefile style using `g++ -M -fmodules 1.cpp`, but that's very confusing, and other compilers don't have it.

  * To use the standard `P1689R5` format, use `g++ 1.cpp -M -fmodules -fdeps-format=p1689r5 -fdeps-file=1.txt -fdeps-target=a.o`. The `fdeps-target=a.o` is optional, and justs the output filename reported in the result.

    This sadly prints some garbage to stdout, which must be discarded.

* Clang:
