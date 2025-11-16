# How to compile libraries manually using CMake?

If you have [determined the build system](/tooling/articles/using_libraries_compiling_manually.md#determine-and-use-the-build-system) to be CMake, follow this procedure.

## Ensure you have CMake installed

Install CMake in MSYS2 using **`pacman -S mingw-w64-clang-x86_64-cmake`**.

Even if you have already installed CMake outside of MSYS2, it's better to install and use MSYS2's version, because it will use the MSYS2's compiler by default, whereas the official CMake for Windows will try to use MSVC (the Visual Studio compiler) by default.

(⚠ This installation command assumes you've been following the previous chapters of the tutorial as is. If you instead configured MSYS2 in some other manner, you might need a different version of CMake, since MSYS2 provides multiple. Consult [this page](./variations/determining_msys2_env.md) for more details, or follow the tutorial from the beginning, reinstalling everything exactly as recommended.)

## Run the "configuration" step

First you might want to `cd` to the directory where you have the source code, purely for convenience. Or perhaps to its parent directory.

Then run the following command:

```sh
cmake -S MySourceDir -B MyBuildDir -DCMAKE_INSTALL_PREFIX=MyInstallDir -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON
```

Where:
* `MySourceDir` is the library source directory containing `CMakeLists.txt`.
* `MyBuildDir` is a temporary directory for the compilation. It will be created if it doesn't exist, and normally it shouldn't exist beforehand.
* `MyInstallDir` is the directory where you want to install the resulting library. It also will be created automatically if it doesn't exist (by other commands later, not immediately). It also shouldn't exist beforehand, unless you're installing several different libraries to the same directory.

**NOTE:** If you need to re-run this command, delete `MyBuildDir` first (e.g. using `rm -rf MyBuildDir` when in MSYS2 terminal), to make sure any changed settings apply correctly.

**NOTE:** None of the `-DCMAKE_...` flags are strictly necessary, but the above are good defaults. I explain their meaning below.

For example, I have extracted the OpenAL source to `C:\code` (so I have their `CMakeLists.txt` at `C:\code\openal-soft-1.23.1\CMakeLists.txt`), so I run following:
```sh
cd C:/code
cmake -S openal-soft-1.23.1 -B openal_build -DCMAKE_INSTALL_PREFIX=openal_install -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON
```
I'm using relative paths here, but you could use e.g. `C:/code/openal-soft-1.23.1` instead of `openal-soft-1.23.1`, and so on.

On success it should eventually print something like this:
```
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: C:/code/openal_build
```

And the directory you specified in `-B ...` will get created. Not the installation directory yet, though.

You can look through the printed logs to see if you get any warnings, such as ones about [missing dependencies](/tooling/articles/using_libraries_compiling_manually.md#install-dependencies). You don't *have* to fix them immediately if the command above succeeds. Try to continue with this procedure and see if everything works fine or not.

**NOTE:** If your library [depends on other libraries you've installed manually](/tooling/articles/using_libraries_compiling_manually.md#install-dependencies), you can specify the same installation directory for all of them, which should make CMake find them automatically. (It's also possible keep the installation directories separate, use `-DCMAKE_PREFIX_PATH=...` to give CMake a list of directories to search, e.g. on Windows `-DCMAKE_PREFIX_PATH="C:\foo;C:\bar"` or `"/c/foo:/c/bar"`, and on other platforms always use the `;` separator)

Additional information:

<details><summary><b>The <code>-DCMAKE_...</code> flags</b></summary>

The *minimal* usage of CMake is just `cmake -S ... -B ...`. (You can even skip either `-S ...` or `-B ...` if it matches the current directory. You'll also see people `cd`ing to the build directory and doing `cmake MySourceDir`, which also works. If the source directory is a parent of the build directory, you'll see people do `cmake ..`.)

The other flags I've recommended above are:

* `-DCMAKE_INSTALL_PREFIX=...` will install the resulting library to the specified directory. Not specifying this will use the default directory, which is a bad idea on Windows:

  * In MSYS2 it defaults to `C:\msys64\clang64`, which is bad, because libraries installed to that location can conflict with libraries installed via `pacman`.

  * On Windows outside of MSYS2, it defaults to `Program Files`, which makes zero sense for installing libraries.

  * On Linux it defaults to `/usr/local`, which is an otherwise empty directory made specifically for installing user-built libraries/apps. This is fine-ish, but it can be hard to keep track of what you have installed there, and hard to remove individual libraries/apps without deleting this entire directory.

   Another option is to not install anywhere (skip the `cmake --install` step below), and manually copy the files, then you don't care about this flag. But this is stone age technology.

* `-DCMAKE_BUILD_TYPE=Release` will perform a release build (with optimization enabled, and with no debugging information). Some other modes are `Debug` and `RelWithDebInfo` (both enables optimizations and debug information).

* `-DBUILD_SHARED_LIBS=ON` will enable building [shared libraries](/tooling/articles/using_libraries_pacman.md#step-2-make-sure-calling-functions-works), because some libraries don't enable them by default. This isn't strictly necessary.

</details>

<details><summary><b>Library-specific options</b></summary>

Libraries often expose custom CMake options that you can tweak. Add `-LH` to the command above to print a list of those options. (If you just want to see the list, you don't have to delete the build directory first, and can just re-run the same command with `-LH` on an existing build directory.)

If you want to change some of those options, do so via `-D...=...`. For this, delete the build directory first.

CMake also has a GUI that you can use to view those options, though it's just a fancy replacement for `-LH`. You can install it via `pacman -S mingw-w64-clang-x86_64-cmake-gui`, and run via `cmake-gui MyBuildDir` (or without the build directory).

[![cmake gui](/tooling/images/cmake_gui_openal.png)](/tooling/images/cmake_gui_openal.png)

</details>

## Build the library

After ["configuring" the library](#run-the-configuration-step), you can build it.

Use `cmake --build MyBuildDir`, where `MyBuildDir` is the build directory that you specified above.

Wait a few minutes for the compilation to finish.

If you edit the library's source code, you normally only need to rerun this command, not the previous one (`cmake -S ... -B ...`).

Some useful options::

* If you add the `--verbose` flag, you'll be able to see the specific compiler commands that are being executed.

* CMake can run several instances of the compiler in parallel to speed up compilation. In MSYS2 this is enabled by default.

  You can control this via `-j...`, e.g. `-j4` to run 4 instances in parallel. A good default value is the your number of CPU cores. There are some generators (see below) default to `-j1`, so sometimes you need to set this explicitly.

Additional information:

<details><summary><b>Generators</b></summary>

CMake doesn't run the compiler directly. Instead, it generates a project for some other build system (when you do `cmake -S ... -B ...`), and then launches that build system (when you do `cmake --build ...`).

There are several to choose from, CMake calls them "generators". You can choose one by padding `-G...` to the initial `cmake -S ... -B ...` command. You can view the available generators and the default by running `cmake --help`, they are printed at the very end.

In MSYS2 CMake the default generator is Ninja (as if you passed `-GNinja`). This is the most sensible option.

On Linux the default is often Make (`-G"Unix Makefiles"`), which is alright, but unlike Ninja it defaults to `-j1` (see above), so if you want resonable build times there, you have to either switch to Ninja using `-GNinja`, or manually pass `-j...`.

</details>

## Install the library

After finishing the previous step, the library is already compiled and can in theory be used as is (you'll find all the [`.dll`, `.a`, `.dll.a` files](/tooling/articles/using_libraries_pacman.md#look-at-what-you-have-installed) in the build directory you have specified).

But it's convenient to perform the "installation" step after that, which copies all useful files into one place, so you won't need to look for them and could delete the build directory which would no longer be needed.

Run
```sh
cmake --install MyBuildDir
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

Now if you open some source file (e.g. `core/ambdec.cpp`), you should immediately see some red squiggles, headers not being found, etc. You can read more about why this happens [here](TODO_link).

Note that I'm assuming you [have Clangd installed in VSC](/tooling/articles/configuring_code_completion.md), which you should have if you've been following this tutorial.

In this case, to tell Clangd where the missing headers are (and more), we need to generate a file called `compile_commands.json`, which contains compiler flags used for every source file in the library (which among other things contain the include directories that need to be searched).

Luckily CMake knows how to generate this file.

Re-run the [`cmake -S ... -B ...` command](#run-the-configuration-step) again, but this time add `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` at the end.

And **make sure** the directory you pass to `-B` is same as the source directory plus `/build` at the end, e.g. `cmake -S openal-soft-1.23.1 -B openal-soft-1.23.1/build ...`.

This will generate `compile_commands.json` in the `build` directory, even without running the `cmake --build ...` step.

The reason why we use this specific directory is because that's where Clangd searches for the JSON by default (in addition to the current directory itself). There are ways to configure it to use any other directory, but it's easier to respect the default for now.

Now you might need to press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` to nudge Clangd to search for the JSON again, and the red squiggles should be gone.

That's all. Now the code navigation should work correctly and there should be no red squiggles.
