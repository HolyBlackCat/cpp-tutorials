# How to use libraries?

## What is a library?

A library is a collection of functions, classes, etc, that you can add to your program and use.

Some libraries simplify things that you could just do yourself (various computations, file operations). Some let you do things you couldn't do otherwise (graphics, sound) (those are usually provided by the operating system vendor, or use such a library under the hood).

A **"third-party library"** refers to any library than's not the standard library, and not provided with your OS/compiler by default. I.e. a library that some random guy made and uploaded to the Internet.

## How do I use a library?

Here's a rough outline:

1. Download its source code.
2. Compile it to produce a `.dll` and/or `.a`.
3. `#include` the library header in your source code.
4. When compiling your code, "link" the library (specify a compiler flag, telling it to use that `.dll` or `.a`).

Package managers (such as MSYS2's `pacman`) take care of steps 1 and 2. They either download a library already compiled by someone else (MSYS2's `pacman` does this) or compile it automatically for you (`vcpkg` does this). So most of the time you should check

Some libraries are "header-only", they don't require steps 2 and 4. You just download the header and include it (package managers then just automate downloading it).

## A specific example

Let's go over the procedure for a specific library. I'll use "OpenAL-soft" as an example (sometimes called just "OpenAL", "AL" stands for "audio library").<br/>
It's a library that can play sounds, and is often used for games.

Even if you don't need this specific library, I suggest installing it for practice, before moving to the libraries you actually need.

## 1. [Try installing the library in MSYS2's `pacman`](/using_libraries_pacman.md)

If the library is not in `pacman`, or if you want to practice compiling it manually, see below. You can also try other package managers (Conan or Vcpkg).

## 2. [Or compile the library manually](/using_libraries_compiling_manually.md)
