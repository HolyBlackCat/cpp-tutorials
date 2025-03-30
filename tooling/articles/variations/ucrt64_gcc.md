# Variant: GCC in MSYS2 UCRT64

This is a small modification to the tutorial you're about to read.

The tutorial recommends the Clang compiler, but you will install the GCC compiler instead.

Do the following during the tutorial:

* Any time the tutorial mentions `MSYS2 CLANG64` in any context, use `MSYS2 UCRT64` instead.

* Any time the tutorial mentions the `C:\msys64\clang64` directory, use `C:\msys64\ucrt64` instead.

* Any time the tutorial mentions `mingw-w64-clang-x86_64-...` (in `pacman` commands), use `mingw-w64-ucrt-x86_64-...` instead.

* Any time the tutorial mentions `clang` or `clang++`, use `gcc` or `g++` respectively instead.

---

Let's test your understanding.

If the tutorial tells you to type `pacman -S mingw-w64-clang-x86_64-clang`, what are you going to type instead? The correct answer is `pacman -S mingw-w64-ucrt-x86_64-gcc`.
