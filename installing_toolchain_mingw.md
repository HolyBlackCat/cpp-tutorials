# Installing a toolchain on Windows

## Choosing a toolchain

There are two prominent toolchains on Windows:

* The one bundled with Visual Studio (not Visual Studio Code), using **MSVC** compiler.
* A loose group of toolchains called **MinGW**, which uses the same tools you'll commonly find on Linux.

We'll be installing the later, with the Clang compiler.

> [Why MinGW rather than MSVC? And why Clang rather than GCC?](/why_mingw.md)

More specifically, we'll be installing MinGW using MSYS2.

> [What is MSYS2? Why MSYS2?](/why_msys2.md)
