# What is the difference between `make` and `mingw32-make`?

In MSYS2 you can find two different "make" programs:

* `make`, installed by `pacman -S make`.
* `mingw32-make`, installed by `pacman -S mingw-w64-clang-x86_64-make` ([or any other `mingw-...-make`](TODO_msys2_envs)).

Normally the rule of thumb is to not install anything with the name not prefixed with `mingw-w64-clang-x86_64-` ([or the prefix of your choice](TODO_msys2_envs)) when the prefixed alternative is available. But in the case of Make, both options are viable.

## Difference between the two

* `mingw32-make` has its path-manipulating functions operate on Windows-style paths, while `make` uses Linux-style paths.

* `mingw32-make` is a lot faster.

* `mingw32-make` has [minor bugs](https://github.com/msys2/MINGW-packages/issues/17735).

* When using it with CMake, `-G "MSYS Makefiles"` corresponds to `make`, and `-G "MinGW Makefiles"` corresponds to `mingw32-make`.

I recommend `make` as the default choice, to have easy compatibility between Windows and Linux makefiles.

If the performance becomes a problem, then you can tweak your makefiles (if needed) to support `mingw32-make` as well.

## The default shell

Make runs the commands using [a shell](/terminal_for_dummies.md). It used to be that `mingw32-make` used `cmd` shell while `make` used `sh` shell, which used to be an additional source of incompatibility.

It's no longer the case in MSYS2, both now use `sh`.

If you download `mingw32-make` from a source other than MSYS2, you can end up with a version that still defaults to `cmd`. (If that's actually what you want, you can switch MSYS2's version to a different shell by setting the `SHELL` variable in the makefile.)
