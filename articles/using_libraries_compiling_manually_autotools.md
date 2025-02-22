# How to compile libraries manually using Autotools?

It becomes increasingly hard to find libraries that can only be built with Autotools and don't support more modern build system.

But every once in a while you'll find one. Or you'll find one that can be built with both, but with broken CMake support.

## Download a test library

I'll be using [FreeType](https://freetype.org/download.html) for this demonstration, since OpenAL that I've used for the CMake demo doesn't support Autotools.

Download the source code from the link above (click `Download`, then click on the latest version, e.g. `freetype-2.13.3.tar.gz` at the time of writing).

`.tar.gz` is an archive format, not unlike `.zip`. Modern Windows should be able to work with it by default; if not, install [7zip](https://www.7-zip.org/).

## Make sure you have `make` installed

```sh
pacman -S make
```
(Note that we're installing [`make` and not `mingw-w64-clang-x86_64-make`](/articles/different_flavors_of_make.md), though the latter could work too, and having both isn't a problem.)

## Create a build directory and `cd` to it

For example, I unzipped the source to `C:\code\freetype-2.13.3`, created an empty directory at `C:\code\freetype_build`, and `cd`ed to it.

```bash
cd /c/code
mkdir freetype_build
cd freetype_build
```


## Run `configure`

This needs to be done in the MSYS2 terminal (or at least in some form of [Bash](/articles/terminal_for_dummies.md#what-is-a-shell)).

From the build directory (without `cd`ing to the source directory), run

```
path/to/configure --prefix=MyInstallDir
```
Where `MyInstallDir` is the desired installation directory.

E.g. I ran `../freetype-2.13.3/configure --prefix=/c/code/freetype_install`. (Here `..` refers to the parent directory, so this is equivalent to `/c/code/freetype-2.13.3/configure --prefix=/c/code/freetype_install`.)

You'll often see people not create a separate build directory, and run `./configure` directly from the source directory. This works, but then you'll have the temporary build files all over the source code, which works, but probably isn't the best idea.

## Build the library

To compile the library, run `` make -j`nproc` `` and wait.

[`-j` has the same meaning as with CMake.](/articles/using_libraries_compiling_manually_cmake.md#build-the-library)

## Install the library

[Like with CMake](/articles/using_libraries_compiling_manually_cmake.md#install-the-library), there's an optional (but recommended) installation step after building the library.

Run `make install`, and everything should be installed to the directory that you [passed to `--prefix=...`](#run-configure).

You can delete the build directory, as it shouldn't be needed anymore. And the source directory too, if you want.

---

For using the resulting library, proceed back to [this](/articles/using_libraries_compiling_manually.md#determine-the-compiler-flags).

---

## Looking at the library source code in VSC

If you now try to look at the library source code, you might see some red squiggles.

Normally this is solved by generating a file called [`compile_commands.json`](TODO_link), but in this case we're out of luck because the tool used to generate it for Autotools (`bear`) only works on Linux, and MSYS2 doesn't have a port of it.

You can try making a small program that generates one based on the output of `make` (look at what CMake generates for inspiration). Since Autotools is so rarely used nowadays, this is left as an exercise to the reader.
