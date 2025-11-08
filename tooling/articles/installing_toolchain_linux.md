# Installing a toolchain on Linux

This tutorial is written primarily for Windows, but all the same tools are available on Linux, so following it on Linux should be easy.

## Choosing a toolchain

The two popular compilers on Linux are **Clang** and **GCC**. I have a [preference towards Clang](./choosing_compiler_and_more.md#choosing-a-compiler), but you can use whatever you like more.

Install either `clang` or `gcc` from your package manager.

**Note to Ubuntu and Debian users:** The version of Clang in the standard repositories if often outdated, especially if you're using an LTS distro. Consider installing the latest Clang from https://apt.llvm.org/.

If you chose GCC instead of Clang, then any time the tutorial tells you to run `clang++`, run `g++` instead.

The executables on Linux don't have the `.exe` extension, unlike on Windows, so any time the tutorial tells you to do something with `blah.exe`, you instead should use just `blah`.

## MSYS2

The tutorial uses MSYS2 on Windows to install the compiler and other tools.

MSYS2 is Windows-only, but you don't need it on Linux, just install everything from your package manager.

When the tutorial tells you to run something in the MSYS2 terminal, run it in your normal terminal instead.

When the tutorial tells you to install something by running `pacman -S mingw-w64-clang-x86_64-...`, install it from your package manager instead, e.g. using `sudo apt install ...` on Ubuntu/Debian. Note the lack of the `mingw-w64-clang-x86_64-` prefix, which is something MSYS2-specific.

## PATH

On Linux you won't have to adjust `PATH` as explained in the chapter [Terminal for Dummies](./terminal_for_dummies.md). The tools you install from your package manager should be avaiable by default, without any `PATH` changes.
