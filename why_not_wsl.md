# Why do I not recommend learning to code in [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)?

To preface, there's nothing wrong with WSL, and it has its uses (running certain Linux tools not available on Windows).

However, I don't think it should be your primary development tool if you're just starting, for one reason:

* Your goal as a programmer is to create programs for other people to use.

Compiling in WSL produces Linux executables, that you can't run on Windows without WSL. You can't expect an average user to install it.

If you make a little game, your friends won't be able to play it without messing with WSL themselves.

## Cross-compilation

It is possible to "cross-compile" (compile executables for one OS while using another, e.g. Windows executables in WSL Linux), but it is not trivial, and is definitely harder than compiling directly on Windows.

## MSYS2

If your reason for using WSL is to have access to command-line tools from Linux (`bash`, `grep`, `sed`...), consider that [MSYS2](/why_msys2.md) has most of them available.
