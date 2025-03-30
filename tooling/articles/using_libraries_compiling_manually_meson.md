# How to compile libraries manually using Meson?

If you have [determined the build system](/tooling/articles/using_libraries_compiling_manually.md#determine-and-use-the-build-system) to be Meson, follow this procedure.

## Download a test library

I'll be using [FreeType](https://freetype.org/download.html) for this demonstration, since OpenAL that I've used for the CMake demo doesn't support Autotools.

Download the source code from the link above (click `Download`, then click on the latest version, e.g. `freetype-2.13.3.tar.gz` at the time of writing).

`.tar.gz` is an archive format, not unlike `.zip`. Modern Windows should be able to work with it by default; if not, install [7zip](https://www.7-zip.org/).

## Ensure you have Meson installed

Install Meson in MSYS2 using **`pacman -S mingw-w64-clang-x86_64-meson`**.

Even if you have already installed Meson outside of MSYS2, it's better to install and use MSYS2's version, to avoid any possible issues with compatibility.

## Run the "configuration" step

[If you're familiar with CMake](/tooling/articles/using_libraries_compiling_manually_cmake.md), you might notice that the procedure is very similar.

First you might want to `cd` to the directory where you have the source code, purely for convenience. Or perhaps to its parent directory.

Then run the following command:

```sh
meson setup MyBuildDir MySourceDir --prefix=MyInstallDir
```
Here `MySourceDir` is the library source directory containing `meson.build`, `MyBuildDir` is a temporary directory for the compilation, and `MyInstallDir` is the directory where you want to install the resulting library.

I also recommend adding `--buildtype=release`, and `--prefix=...` isn't strictly necessary, more on that below.

**NOTE:** If your library [depends on other libraries you've installed manually](/tooling/articles/using_libraries_compiling_manually.md#install-dependencies), you can specify the same installation directory for all of them, which should make Meson find them automatically. There are other solutions to this (to keep them separate), but this one is the easiest.

For example, I have extracted the FreeType source to `C:\code` (so I have their `meson.build` at `C:\code\freetype-2.13.3\meson.build`), so I run following:
```sh
cd C:/code
meson setup freetype_build freetype-2.13.3 --prefix=C:/code/freetype_install --buildtype=release
```
I'm using relative paths here, but you could use e.g. `C:/code/freetype-2.13.3` instead of `freetype-2.13.3`, and so on.

On success, you should see the following after a while:

[![meson finished configuring freetype](/tooling/images/meson_finished_configuring_freetype.png)](/tooling/images/meson_finished_configuring_freetype.png)

(I've also got some unimportant errors, but it worked anyway.)

This creates build directory you specified in `meson setup ...`. Not the installation directory yet, though.

You can look through the logs to see if you get any warnings, such as ones about [missing dependencies](/tooling/articles/using_libraries_compiling_manually.md#install-dependencies). Though you don't *have* to fix them immediately if the command above succeeds. Try to continue with this procedure and see if everything works fine or not

<details><summary><b>More about Meson flags</b></summary>

The *minimal* usage of Meson is just `meson MyBuildDir`. People usually `cd` to the source directory, then it doesn't need to be specified.

I've also thrown in a few more useful flags:

* `--prefix=...` will install the resulting library to the specified directory. Not specifying this will make it default to `C:\msys64\clang64`, which isn't great, because libraries installed to that location can conflict with libraries installed via `pacman`. (This is a problem unique to MSYS2, because on Linux the default installation directory `/usr/local` is separate from everything else and is empty by default.)

   Another option is to not install anywhere, and manually copy the files (then you don't care about this flag), but this is stone age technology.

   If you're [familiar with CMake](/tooling/articles/using_libraries_compiling_manually_cmake.md), this option is equivalent to CMake's `-DCMAKE_INSTALL_PREFIX=...`.

* `--buildtype=release` will perform a release build (with optimization enabled, and with no debugging information). Change this if you want to.

   If you're [familiar with CMake](/tooling/articles/using_libraries_compiling_manually_cmake.md), this option is equivalent to CMake's `-DCMAKE_BUILD_TYPE=Release`.

</details>


## Build the library

After ["configuring" the library](#run-the-configuration-step), you can build it.

Use `meson compile -C MyBuildDir`, where `MyBuildDir` is the build directory that you specified above.

The `-C ...` part isn't necessary if you `cd` to this directory first.

[Like CMake](/tooling/articles/using_libraries_compiling_manually_cmake.md#build-the-library), this supports the `-j...` flag, but unlike CMake it has a reasonable default value, so I don't recommend setting this flag.

Wait a few minutes for the compilation to finish.

If you edit the library's source code, you normally only need to rerun this command, not the previous one (`meson setup ...`).


## Install the library

After finishing the previous step, the library is already compiled and can in theory be used as is (you'll find all the [`.dll`, `.a`, `.dll.a` files](/tooling/articles/using_libraries_pacman.md#look-at-what-you-have-installed) in the build directory you have specified).

But it's convenient to perform the "installation" step after that, which copies all userful files into one place, so you don't need to look for them and can delete the build directory which is no longer needed.

Run
```sh
meson install -C MyBuildDir
```
(Note, we're specifying the build directory, not the installation directory.)

This should populate <code>My<b>Install</b>Dir</code> (which you [specified all the way back](#run-the-configuration-step)) will all the required files ([similar to those](/tooling/articles/using_libraries_pacman.md#look-at-what-you-have-installed)).

You can now delete `MyBuildDir`, as it's no longer needed. And the source directory too, if you don't need it.

---

For using the resulting library, proceed back to [this](/tooling/articles/using_libraries_compiling_manually.md#determine-the-compiler-flags).

---

## Looking at the library source code in VSC

Optionally, if you want to look at the library source code, this part explains how to do so.

Try pressing `File`→`Open Folder`, and open the source directory (`C:\code\openal-soft-1.23.1` in this example).

Now if you open some source file (e.g. `src/autofit/autofit.c`), you should immediately see some red squiggles, headers not being found, etc. You can read more about why this happens [here](TODO_link).

Note that I'm assuming you [have Clangd installed in VSC](/tooling/articles/configuring_code_completion.md), which you should have if you've been following this tutorial.

In this case, for Clangd to understand where the missing headers are (and more), it needs to see the `compile_commands.json` file, which contains compiler flags used for every source file in the library (which among other things contain the include directories that need to be searched).

Meson generates this file automatically, it's in the build directory.

Re-run the [`meson setup ...` command](#run-the-configuration-step) again, but this time set the build directory to the same directory as the source directory, plus `/build` at the end.

E.g. `meson setup freetype-2.13.3/build freetype-2.13.3 ...`. Or if you `cd` to the `freetype-2.13.3` directory first, just `meson setup build ...`, as was explained above.

This will generate `compile_commands.json` in the `build` directory, even without running the `meson compile ...` step.

The reason why we use this specific directory is because that's where Clangd searches for the JSON by default (in addition to the current directory itself). There are ways to configure it to use any other directory, but it's easier to respect the default for now.

Now you might need to press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` to nudge Clangd to search for the JSON again, and the red squiggles should be gone.

That's all, the code navigation should work now.

...**Should** work. I'm still seeing some red squiggles in FreeType, but that's their bug (or poorly written Meson file). [Manunally fixing `compile_commands.json` or writing a `.clangd` config file](TODO_link) to work around this bug is left as an exercise to the reader.
