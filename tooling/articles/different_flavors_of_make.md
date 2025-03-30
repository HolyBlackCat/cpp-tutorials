# What is the difference between `make` and `mingw32-make`?

In MSYS2 you can find two different "make" programs:

* `make`, installed by `pacman -S make`.
* `mingw32-make`, installed by `pacman -S mingw-w64-clang-x86_64-make` ([or any other `mingw-...-make`](/tooling/articles/msys2_environments.md)).

Normally the rule of thumb is to not install anything with the name not prefixed with `mingw-w64-clang-x86_64-` ([or the prefix of your choice](/tooling/articles/msys2_environments.md)) when the prefixed alternative is available. But in the case of Make, both options are viable.

## Difference between the two

* `mingw32-make` has its path-manipulating functions operate on Windows-style paths, while `make` uses Linux-style paths.

* `mingw32-make` is a lot faster.

* `mingw32-make` has [minor bugs](https://github.com/msys2/MINGW-packages/issues/17735).

* When using it with CMake, `-G "MSYS Makefiles"` corresponds to `make`, and `-G "MinGW Makefiles"` corresponds to `mingw32-make`.

## Which one to use?

I recommend `make` for handwritten makefiles, because it's less quirky and makes it easier to write makefile that work both on Linux and Windows.

If the performance becomes a problem, then you can tweak your makefiles (if needed) to support `mingw32-make` as well.

If you were to use Make with something like CMake, then I would recommend `mingw32-make` for better performance. But since `ninja` is a thing, you should be using that with CMake instead.

## The default shell

Make runs the commands using [a shell](/tooling/articles/terminal_for_dummies.md).

Both use `sh` shell by default (same as on Linux), but you can override it manually.

`mingw32-make` falls back on Windows `cmd` if it can't find `sh` in PATH (which should only be possible when running it outside of MSYS2 terminal). And `make` should never be unable to find `sh` because they're installed to the same directory.

I recommend always using `sh` for portability, but forcing `mingw32-make` to use `cmd` could make it a tiny bit faster.

It's possible to detect the current shell (the `SHELL` Make variable is unreliable) using `ifeq ($(shell echo 'foo'),'foo')`, which is true on `cmd`, false on `sh`. So it's possible to support both shells, but I wouldn't bother until performance becomes a problem.
