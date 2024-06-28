# Why do I recommend MSYS2?

There is no single official "MinGW compiler" you can install. Several groups of people provide installers for "MinGW" (with GCC and/or Clang bundled), and you need to choose which one to install.

## The original MinGW vs MinGW-w64

Most of what is called "MinGW" today is actually [MinGW-w64](https://www.mingw-w64.org/), an improved fork of [the original MinGW](https://osdn.net/projects/mingw).

The issues with the original MinGW include:

* No support for multithreading in the standard library (`#include <thread>` and others).
* Can't produce 64-bit executables.
* They also haven't updated their GCC version for a while (still on GCC 9 at the time of writing this, while the latest is GCC 14), so the project seems to be dead.

Everywhere in this tutorial by "MinGW" I mean either the original MinGW or MinGW-w64.

## Different distributions of MinGW-w64

The most popular MinGW-w64 distribution seems to be [MSYS2](https://www.msys2.org/). They provide many different [kinds of compilers](https://www.msys2.org/docs/environments/) (GCC and Clang with different configurations), and [many prebuilt libraries](https://packages.msys2.org/package/). They also provide some command-line [utilities ported from Linux](#msys2-and-cygwin), such as `bash`, `make`, etc.

In my opinion the collection of prebuilt libraries alone makes other distributions impractical, but for completeness here are some popular ones:

* [llvm-mingw](https://github.com/mstorsjo/llvm-mingw) - provides Clang-based toolchain (unclear what's the difference compared to MSYS2).

* [w64devkit](https://github.com/skeeto/w64devkit) - provides GCC (unclear what's the difference compared to MSYS2).

* [GCC with MCF thread model](https://gcc-mcf.lhmouse.com/) - provides a more optimized implementation of standard multithreading primitives ([benchmarks](https://github.com/lhmouse/mcfgthread/wiki/Benchmarking/)).

* [Mingw-builds](https://github.com/niXman/mingw-builds-binaries) - provides some different GCC configurations (including one with MCF thread model, see above). Currently ships a slightly outdated GCC.

* [WinLibs](https://winlibs.com/) - claims to have different philosophy compared to MSYS2, but it's unclear what exactly is the difference. Ships both GCC and Clang, though last time I tried Clang didn't work (had missing DLLs). Doesn't ship libc++.

## MinGW vs Cygwin

[Cygwin](https://www.cygwin.com/) is a different way to use GCC (and other Linux tools) on Windows, unrelated to MinGW.

Cygwin's distinguishing feature is the amount of Linux/POSIX emulation they provide:

* All file paths will appear to your program as e.g. `/cygdrive/c/foo/bar` instead of `C:\foo\bar`.
* They go to great lengths to emulate POSIX functions such as `fork()`, that are not available on Windows otherwise.
* `#ifdef __unix__` returns true and `#ifdef _WIN32` returns false.
* etc

Using this emulation for new projects seems unwise.  It only makes sense if you're porting existing Linux applications to Windows, if they're making heavy use of non-cross-platform POSIX calls. In new applications use WinAPI directly if you need it.

## MSYS2 and Cygwin

MSYS2 uses its own fork of Cygwin for some of the applications they provide (such as `bash`, `pacman`, `make`, `grep`, `sed`...).

They also distribute a version of GCC that you can use to compile your own applications using their Cygwin fork. I don't recommend this for new projects though, for [the same reason I don't recommend Cygwin itself](#mingw-vs-cygwin). (To be clear, not all of the GCC variants they provide use Cygwin. Only one of them does.)

If you don't like Cygwin for some reason, the only Cygwin-based application in MSYS2 you have to use is the package manager (`pacman`). After you install the compiler (and anything else you need), you can use it independently of anything Cygwin-related.
