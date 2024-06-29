# Installing a compiler and a toolchain

### What is a toolchain?

A toolchain is a collection of programs you need to convert the source code you write (which is just text) to an executable (a program you can actually run).

There are several different toolchains, we'll be installing one of them.

### What is a compiler?

A compiler is one of the programs included in a toolchain, the most prominent part of it. It understands the source code you write, and translates it to something a computer can understand.

The three most popular C/C++ compilers today are Clang, GCC, and MSVC.

### What isn't a part of a toolchain?

Programs such as Visual Studio Code, Visual Studio, CLion, XCode... are not parts of a toolchain. While they are convenient, you don't have to use them. And when you do, you can usually choose one independently of your toolchain.

Those are called IDEs, or "integrated development environments". "Integrated" means they combine together different features useful for programming.

We will make the first steps without an IDE, then install one later.

## How do I install a toolchain?

The process depends on your operating system:

* ### [I'm using Windows](/installing_toolchain_mingw.md)
* ### [I'm using Linux](TODO) (TODO: link why_not_wsl.md from the linux page)
