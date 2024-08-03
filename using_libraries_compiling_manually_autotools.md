# How to compile libraries manually using Autotools?

It becomes increasingly hard to find a library that can't be built with CMake, and can only be built with Autotools. (Or to find one that can be built with Autotools at all.)

But every once in a while you'll find one. Or you'll find one that can be built with both, but with broken CMake support.

## Download and unzip the source code

This speaks for itself.

I'll be using [FreeType](https://freetype.org/download.html) for this demonstration, since OpenAL that I've used before doesn't support Autotools.

Download the source code from the link above (from there, click https://savannah.nongnu.org/download/freetype/, then click on the latest version, e.g. `freetype-2.13.2.tar.gz` at the time of writing).

`.tar.gz` is an archive format, not unlike `.zip`. Modern Windows should be able to work with it by default; if not, install [7zip](https://www.7-zip.org/).

## Make sure you have `make` installed

```sh
pacman -S make
```
(Note that we're installing [`make` and not `mingw-w64-clang-x86_64-make`](/different_flavors_of_make.md), though the latter could work too, and having both isn't a problem.)

## Create a build directory and `cd` to it

For example, I unzipped the source to `C:\code\freetype-2.13.2`, created an empty directory at `C:\code\freetype_build` and did `cd C:/code/freetype_build`.

## Run `configure`

From the build directory (without `cd`ing to the source directory), run

```
path/to/configure --prefix=MyInstallDir
```
Where `MyInstallDir` is the desired installation directory.

E.g. I ran `C:/code/freetype-2.13.2/configure --prefix=C:/code/freetype_install`.

You'll often see people not create a separate build directory, and run `./configure` directly from the source directory. This works, but then you'll have the temporary build files all over the source code, which works, but probably isn't the best idea.

## Build the library

To compile the library, run `` make -j`nproc` `` and wait.

[`-j` has the same meaning as with CMake.](/using_libraries_compiling_manually_cmake.md#build-the-library)

## Install the library

[Like with CMake](/using_libraries_compiling_manually_cmake.md#install-the-library), there's an optional (but recommended) installation step after building the library.

Run `make install`, and everything should be installed to the directory that you [passed to `--prefix=...`](#run-configure).

You can delete the build directory, as it shouldn't be needed anymore. And the source directory too, if you want.

---

For using the resulting library, proceed back to [this](/using_libraries_compiling_manually.md#determine-the-compiler-flags).
