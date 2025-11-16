# How to compile programs consisting of multiple source files?

Everything explained here works with both Clang and GCC compilers. The examples use `clang++` because that's what the rest of the tutorial uses, but you can use `g++` if you've been using the GCC compiler so far.

## Why multiple files?

Larger programs are split into multiple source files, for better organization and for faster compilation times (to only recompile the files that have changed).

## A simple test program

Your C++ book should explain how to correctly split a program into multiple files.

This is a simple test program (consisting of three files) so you have something to test the instructions below on:

**`2.h`**
```cpp
#pragma once

void foo();
```
**`2.cpp`**
```cpp
#include "2.h"
#include <iostream>

void foo()
{
    std::cout << "Hello!\n";
}
```
**`1.cpp`**
```cpp
#include "2.h"

int main()
{
    foo();
}
```

Note that we **do not** `#include` one `.cpp` file into another. This is almost always wrong. Include header files (normally `.h` or `.hpp`). Your C++ book should explain more about this.

## The simplest approach

If you try to compile using `clang++ 1.cpp` or `clang++ 2.cpp`, you'll get errors about `foo()` or `main()` being missing respectively.

(If you instead get errors about `2.h` not being found, it's probably not in the same directory as the `.cpp` files. Either move it there, or pass the directory where it's located to `-I` if you prefer to keep it separate.)

The easiest fix is to pass **all** the `.cpp` files to the compiler command: `clang++ 1.cpp 2.cpp -o prog`. Try it, make sure it works.

You only need to list the `.cpp` files (or `.c` for C), not `.h` or `.hpp`.

You can use this approach while you're still learning, but the issue with it is that it's **slow**:

* Firstly because if you only change one `.cpp` file, this rebuilds all of them, which isn't viable for large projects.

* Secondly because you could build multiple `.cpp` files in parallel, and this approach doesn't do this.

## The proper procedure

Because of the issues listed above, most programs are built using a different approach. Like this:

```sh
clang++ 1.cpp -c -o 1.o
clang++ 2.cpp -c -o 2.o
clang++ 1.o 2.o -o prog
```
First we **compile** each individual `.cpp` file into something called "object files" (`.o`).

Notice that we're using the **`-c`** flag to produce object files, without it the compiler would try to produce an executable.

As the last step we **link** the object files into one executable. Internally `clang++` calls a "linker" program to do that.

The **entire process** consisting of compilation and linking is called **"building"** the program. (Though people will often say "compilation" to refer to the entire process.)

Try this, make sure this works on your program.

### Compiler and linker flags

You should have already learned some optional compiler settings or "flags", such as [`-g`](/tooling/articles/debugging_in_terminal.md) or [others](/tooling/articles/recommended_compiler_flags.md). How should we use them in this multi-step procedure?

When in doubt, add them to both the compilation and linking commands. If some flags are only needed in one place, Clang should tell you so.

`-l` and `-L` are only needed when linking, and `-D`,`-I` are only needed when compiling. If you don't know what they are, you can ignore this part.

### Why is this better?

If you just run all three commands manually, this isn't any better than the [single-step procedure](#the-simplest-approach) explained before.

However! This lets you fix both issues mentioned above:

* You can re-run the compilation commands only for those `.cpp` files that you have changed since the last build (and then run the linking command).

* You can run multiple compilation commands at the same time (and after all of them are finished, run the linking command).

However, doing this manually is tedious. That's why we use...

## The build systems

The build systems are programs that run the compiler for your. They keep track of what `.cpp` files have changed to only recompile those, and they can also be configured to pass certain compiler flags automatically, so you don't have to type them every time.

Some of the most popular build systems are:

* **Make** — very barebones and because of that isn't used often. And for the same reason it's great for learning purposes. (You can also hear it being referred to as "GNU Make", as opposed to some other inferior versions.)

* **CMake** — easily the most popular build system for C++, but also often hated for being too complex. Regardless of what you think about it, you should at least learn the basics of it.

* **Meson** — a relatively new contender, slowly gaining popularity.

This tutorial will first explain how to use Make, to give a better understanding of the underlying process, then show how to use the other two build systems.

* ### [Using Make](/tooling/articles/build_system_make.md)
* ### [Using CMake](TODO_cmake)
* ### [Using Meson](TODO_meson)
