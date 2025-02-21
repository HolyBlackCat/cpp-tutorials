# How to compile libraries manually using CMake?

If you have [determined the build system](/using_libraries_compiling_manually.md#determine-and-use-the-build-system) to be CMake, follow this procedure.

## Ensure you have CMake installed

Install CMake in MSYS2 using **`pacman -S mingw-w64-clang-x86_64-cmake`**.

Even if you have already installed CMake outside of MSYS2, it's better to install and use MSYS2's version, because it will use the MSYS2's compiler by default, whereas the official CMake for Windows will try to use MSVC (the Visual Studio compiler) by default.

Also install Ninja: **`pacman -S mingw-w64-clang-x86_64-ninja`**

## Run the "configuration" step

First you might want to `cd` to the directory where you have the source code, purely for convenience. Or perhaps to its parent directory.

Then run the following command:

```sh
cmake -S MySourceDir -B MyBuildDir -DCMAKE_INSTALL_PREFIX=MyInstallDir
```

Here `MySourceDir` is the library source directory containing `CMakeLists.txt`, `MyBuildDir` is a temporary directory for the compilation, and `MyInstallDir` is the directory where you want to install the resulting library.

**NOTE:** Any time you re-run this command, delete `MyBuildDir` first (e.g. using `rm -rf MyBuildDir` when in MSYS2 terminal), to make sure any changed settings apply correctly.

I also recommend adding `-DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON`, and `-DCMAKE_INSTALL_PREFIX=...` isn't strictly necessary, more on that below.

**NOTE:** If your library [depends on other libraries you've installed manually](/using_libraries_compiling_manually.md#install-dependencies), you can specify the same installation directory for all of them, which should make CMake find them automatically. There are other solutions to this (to keep them separate), but this one is the easiest.

For example, I have extracted the OpenAL source to `C:\code` (so I have their `CMakeLists.txt` at `C:\code\openal-soft-1.23.1\CMakeLists.txt`), so I run following:
```sh
cd C:/code
cmake -S openal-soft-1.23.1 -B openal_build -DCMAKE_INSTALL_PREFIX=openal_install -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON
```
I'm using relative paths here, but you could use e.g. `C:/code/openal-soft-1.23.1` instead of `openal-soft-1.23.1`, and so on.

On success, you should see this after a while:
```
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: C:/code/openal_build
```
And the directory you specified in `-B ...` will be created. Not the installation directory yet, though.

You can look through the logs (as printed by `cmake ...`) to see if you get any warnings, such as ones about [missing dependencies](/using_libraries_compiling_manually.md#install-dependencies). Though you don't *have* to fix them immediately if the command above succeeds. Try to continue with this procedure and see if everything works fine or not

Additional information:

<details><summary><b>What's with all the CMake flags?</b></summary>

The *minimal* usage of CMake is just `cmake -S ... -B ...` (and even `-S ...` can be omitted if it's the current directory). You'll also see people `cd`ing to the build directory and doing `cmake ..`, which is equivalent.

I've also thrown in a few more useful flags:

* `-DCMAKE_INSTALL_PREFIX=...` will install the resulting library to the specified directory. Not specifying this will make it default to `C:\msys64\clang64`, which isn't great, because libraries installed to that location can conflict with libraries installed via `pacman`. (This is a problem unique to MSYS2, because on Linux the default installation directory `/usr/local` is separate from everything else and is empty by default.)

   Another option is to not install anywhere, and manually copy the files (then you don't care about this flag), but this is stone age technology.

* `-DCMAKE_BUILD_TYPE=Release` will perform a release build (with optimization enabled, and with no debugging information). Change this if you want to.

* `-DBUILD_SHARED_LIBS=ON` will typically enable building [shared libraries](/using_libraries_pacman.md#step-2-make-sure-calling-functions-works), because some libraries don't enable this by default. This isn't strictly necessary.

</details>

<details><summary><b>CMake GUI</b></summary>

CMake has a GUI that you can use to view library's compilation settings. (Sadly the version of cmake-gui in `pacman` appears to be broken, you have to install it from the official CMake installer.)

[![cmake gui](/images/cmake_gui_openal.png)](/images/cmake_gui_openal.png)

If you find any interesting settings you want to change, delete the build directory and re-run the `cmake ...` command above with the extra `-D...=...` flags.

</details>

## Build the library

After ["configuring" the library](#run-the-configuration-step), you can build it.

Use `` cmake --build MyBuildDir -j`nproc` ``, where `MyBuildDir` is the build directory that you specified above.

The `` -j`nproc` `` part is optional. It speeds up the compilation by running several instances of the compiler in parallel. You can specify how many copies to run using e.g. `-j4` (where `4` is the number of copies), and using `` -j`nproc` `` will use the number of your CPU cores (run `nproc` on its own to see that number).

Wait a few minutes for the compilation to finish.

If you edit the library's source code, you normally only need to rerun this command, not the previous one (`cmake -S ... -B ...`).

## Install the library

After finishing the previous step, the library is already compiled and can in theory be used as is (you'll find all the [`.dll`, `.a`, `.dll.a` files](/using_libraries_pacman.md#look-at-what-you-have-installed) in the build directory you have specified).

But it's convenient to perform the "installation" step after that, which copies all userful files into one place, so you don't need to look for them and can delete the build directory which is no longer needed.

Run
```sh
cmake --install MyBuildDir
```
(Note, we're specifying the build directory, not the installation directory.)

This should populate <code>My<b>Install</b>Dir</code> (which you [specified all the way back](#run-the-configuration-step)) will all the required files ([similar to those](/using_libraries_pacman.md#look-at-what-you-have-installed)).

You can now delete `MyBuildDir`, as it's no longer needed. And the source directory too, if you don't need it.

---

For using the resulting library, proceed back to [this](/using_libraries_compiling_manually.md#determine-the-compiler-flags).

---

## Looking at the library source code in VSC

Optionally, if you want to look at the library source code, this part explains how to do so.

Try pressing `File`→`Open Folder`, and open the source directory (`C:\code\openal-soft-1.23.1` in this example).

Now if you open some source file (e.g. `core/ambdec.cpp`), you should immediately see some red squiggles, headers not being found, etc. You can read more about why this happens [here](TODO_link).

Note that I'm assuming you [have Clangd installed in VSC](/configuring_code_completion.md), which you should have if you've been following this tutorial.

In this case, to tell Clangd where the missing headers are (and more), we need to generate a file called `compile_commands.json`, which contains compiler flags used for every source file in the library (which among other things contain the include directories that need to be searched).

Luckily CMake knows how to generate this file.

Re-run the [`cmake -S ... -B ...` command](#run-the-configuration-step) again, but this time add `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` at the end.

And **make sure** the directory you pass to `-B` is same as the source directory plus `/build` at the end, e.g. `cmake -S openal-soft-1.23.1 -B openal-soft-1.23.1/build ...`.

This will generate `compile_commands.json` in the `build` directory, even without running the `cmake --build ...` step.

The reason why we use this specific directory is because that's where Clangd searches for the JSON by default (in addition to the current directory itself). There are ways to configure it to use any other directory, but it's easier to respect the default for now.

Now you might need to press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` to nudge Clangd to search for the JSON again, and the red squiggles should be gone.

That's all. Now the code navigation should work correctly and there should be no red squiggles.
