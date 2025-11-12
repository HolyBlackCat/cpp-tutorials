## C++ tooling tutorial

### What is this tutorial for?

* This tutorial explains how to install the *tools* needed to code in C/C++, but not *how* to code. You'll still need a generic book/tutorial on C/C++.

* This tutorial is primarly for Windows. But all tools suggested here are cross-platform, so it's not terribly hard to follow the tutorial on other OSes too (all tools other than MSYS2, which is Windows-only, but isn't necessary on other platforms).

* There are endless possible tooling choices. In this tutorial we'll install the Clang compiler (or GCC if you prefer) using MSYS2; along with Visual Studio Code, Clangd, LLDB (or GDB) and LLDB-DAP. I consider those to be the best choices available.

  I do explain [why I prefer those specific tools](/tooling/articles/why_philosophy.md), and what are the alternatives.

### How to read this tutorial?

I recommend following the steps below **in order**, unless you already have a good idea what you're doing.

If you're new to C/C++, don't binge-read everything at once. Practice writing small programs and read your C/C++ book between the chapters of this tutorial.

### Basics:

* ### [Installing a toolchain](/tooling/articles/installing_toolchain.md)

* ### [How to compile and run programs in the terminal](/tooling/articles/compiling_in_terminal.md)

* ### [Installing and configuring VSC](/tooling/articles/installing_vsc.md)

### Debugging:

* ### [Debugging in the terminal](/tooling/articles/debugging_in_terminal.md)

* ### [Debugging in VSC](/tooling/articles/configuring_vsc_debugger.md)

### Additional information:

(You can read those in any order, but read the Basics above first.)

* ### [Recommended compiler settings](/tooling/articles/recommended_compiler_flags.md)

* ### [Compiling programs with multiple source code files](/tooling/articles/multifile_programs.md)

* ### [Using third-party libraries](/tooling/articles/using_libraries.md)
