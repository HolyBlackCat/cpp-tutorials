# Alternative choices in this tutorial

This tutorial explains how to install Clang in MSYS2 CLANG64. If you don't know what you're doing, just follow the tutorial as is.

But there are some alternative routes you can take:

* GCC instead of Clang:
  * [GCC in MSYS2 UCRT64](./ucrt64_gcc.md) — the default choice for GCC.
  * [GCC in MSYS2 MINGW64](./mingw64_gcc.md) — GCC with older C standard library.

* Clang but in a different way:

  * [Clang in MSYS2 UCRT64](./ucrt64_clang.md) — Clang but in a GCC-centric environment. Makes it easier to switch between GCC and Clang. Lacks the address sanitizer.

  * [Clang in MSYS2 MINGW64](TODO_LINK) — same, but also with the older C standard library.
