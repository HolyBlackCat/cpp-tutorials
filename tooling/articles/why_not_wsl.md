# Why do I not recommend learning to code in [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)?

There's nothing wrong with using WSL to run Linux tools that are otherwise unavailable on Windows (such as Valgrind).

However, I don't think it should be your primary development tool if you're just starting, for one reason:

**Your goal as a programmer is to create programs for other people to use.**

Compiling in WSL produces Linux executables, that you can't run on Windows without WSL. You can't expect an average user to install it.

If you make a little game, your friends won't be able to play it without messing with WSL themselves.

## But WSL makes things easier!

It doesn't. The difference compared to [MSYS2](/tooling/articles/why_msys2.md) is minimal.

You can install the compiler/debugger/etc with a single command in WSL, but the same is true for MSYS2.

MSYS2 has ports of `bash`, `grep`, `sed`, etc, if you want those.

## Cross-compilation

It is possible to "cross-compile" in WSL (compile executables for one OS while using another, e.g. Windows executables while using Linux in WSL), but it is not trivial, and is definitely harder than compiling directly on Windows (or than compiling for Linux in WSL).
