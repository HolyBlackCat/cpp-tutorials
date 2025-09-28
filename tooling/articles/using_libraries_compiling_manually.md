# How to compile libraries manually?

You must already be familiar with [installing libraries via `pacman` and using them](/tooling/articles/using_libraries_pacman.md) (if not, read that page). This page builds upon that one.

## Uninstall the library if you already have it

If you've already installed the same library using `pacman`, I recommend uninstalling it first to avoid confusion. E.g. for OpenAL:

```sh
pacman -R mingw-w64-clang-x86_64-openal
```
Note that `-R` leaves behind the *dependencies* (other libraries that this one needs, that got installed along with it). This is usually undesired (so usually you'd use `-Rs` instead to uninstall them too), but is actually helpful in our case, as those dependencies might be needed to build the library yourself, [more on that below](#install-dependencies).

## Download the source code

Find and download the source code for your library, normally for its latest release.

I'll be using OpenAL-soft as an example again. On https://github.com/kcat/openal-soft, go to `Releases` and grab the `Source code (zip)` for the lastest releases.

Some libraries don't do releases, then just grab the latest version of the code (on Github, the green `<> Code` button → `Download ZIP`).

## Unzip the source code

Unzip the source code somewhere. The path to it shouldn't contain spaces or any special or non-latin characters, it shouldn't be on a network drive nor on a USB stick.

## Install dependencies

Some libraries depend on other libraries to function (properly or at all). Read the documentation for your library to know what other libraries you need to install first.

(A "dependency" refers to a library used by some other library or application.)

If you're can't find anything, or aren't sure, skip this step and see if it works out.

If you do find something, install them using `pacman` or manually using this procedure.

The alternative to reading documentation is, if MSYS2 has a package for the library you're building, to consult [the package information](https://packages.msys2.org/package/mingw-w64-clang-x86_64-openal) for the list of dependencies you need to install: `Dependencies` and `Build Dependencies`. If you already had this library installed, and uninstalled it via `-R` without `s`, the `Dependencies` will already be installed. But some `Build Dependencies` could be missing, so there is no replacement for consuling the MSYS  package list.

**We'll not be installing any dependencies for OpenAL.** Despite them not being mentioned in in the README, it does have some optional dependencies, but seems to work fine without them.

## Determine and use the build system

A "build system" is a tool that automatically invokes the compiler for multiple source files, simplifying the compilation of large projects. You need to determine what build system the library uses, and use it.

To determine it, look at the source code:

* If a file named **`CMakeLists.txt`** exists, the build system is **CMake**.
* If a file named **`meson.build`** exists, the build system is **Meson**.
* If a file named **`configure`** exists, the build system is likely **Autotools** (or follows the same build procedure as it).

Some projects support multiple build systems, you can pick either one.

As you can see, OpenAL uses CMake:

![OpenAL CMakeLists.txt](/tooling/images/openal_cmakelists_txt.png)

* ### [Build using CMake](/tooling/articles/using_libraries_compiling_manually_cmake.md)
* ### [Build using Meson](/tooling/articles/using_libraries_compiling_manually_meson.md)
* ### [Build using Autotools](/tooling/articles/using_libraries_compiling_manually_autotools.md)

## Determine the compiler flags

To use the resulting library, you can [follow the same procedure with `pkg-config` as before](/tooling/articles/using_libraries_pacman.md#determining-compiler-flags-using-pkg-config).

But since the library is installed to a custom location, you need to point `pkg-config` to it. Run following in your MSYS2 terminal:

```sh
PKG_CONFIG_PATH=C:/code/openal_install/lib/pkgconfig PKG_CONFIG_LIBDIR=- pkg-config --libs --cflags openal
```

Replace `C:/code/openal_install` with your installation directory. Notice that there's always `.../lib/pkgconfig` at the end, since that's where `.pc` files are installed. We also set `PKG_CONFIG_LIBDIR` to a junk value to disable searching for libraries in the default directories (`C:\msys64\clang64`), but this is optional.

For me this prints:
```sh
-IC:/code/openal_install/include -IC:/code/openal_install/include/AL -LC:/code/openal_install/lib -lOpenAL32
```

For example, I used `C:\code\openal_install` as the installation directory,

### Guessing the flags

[Like before](/tooling/articles/using_libraries_pacman.md#guessing-the-compiler-flags), you can guess the correct compiler flags if your library doesn't come with a `.pc` file.

Follow the same procedure as linked, but remember that you will also [`-L...`](/tooling/articles/using_libraries_pacman.md#step-2-make-sure-calling-functions-works) (since the `.a` files are installed to a non-default location). And of course [`-I...`](/tooling/articles/using_libraries_pacman.md#step-1-make-sure-include-works) for the headers.
