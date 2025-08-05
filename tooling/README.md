## C++ tooling tutorial

### What is this tutorial for?

* This tutorial explains how to install the *tools* needed to code in C/C++, but not *how* to code. You'll still need a generic book/tutorial on C/C++.

* This tutorial is primarly for Windows. But if you know what you're doing, you can repeat the process on other OSes too (since all the tools we'll install are cross-platform, except MSYS2, which isn't necessary outside of Windows).

* There are endless possible tooling choices. In this tutorial we'll install the **Clang** compiler (or **GCC** if you prefer) using **MSYS2**; and **Visual Studio Code**, **Clangd**, **LLDB** and **LLDB-DAP**, which I consider to be the best choice available.

  I do explain [why I prefer those specific tools](/tooling/articles/why_philosophy.md), and what are the alternatives.

### How to read this tutorial?

I recommend following the steps below **in order**, unless you already have a good idea what you're doing.

If you're new to C/C++, don't binge-read everything at once. Practice writing small programs and read your C/C++ book between the chapters of this tutorial.

### Basics:

* ### [Installing a toolchain](/tooling/articles/installing_toolchain.md)

* ### [How to compile and run programs in the terminal](/tooling/articles/installing_toolchain.md)

* ### [Installing and configuring VSC](/tooling/articles/installing_vsc.md)

### Debugging:

* ### [Debugging in a terminal](/tooling/articles/debugging_in_terminal.md)

* ### [Debugging in VSC](/tooling/articles/configuring_vsc_debugger.md)

### Additional information:

(You can read those in any order.)

* ### [Recommended compiler settings](/tooling/articles/recommended_compiler_flags.md)

* ### [Compiling programs with multiple source code files](/tooling/articles/multifile_programs.md)

* ### [Using third-party libraries](/tooling/articles/using_libraries.md)
