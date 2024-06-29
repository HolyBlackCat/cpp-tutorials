# Installing a toolchain on Windows

## Choosing a toolchain

There are two prominent toolchains on Windows:

* The one bundled with Visual Studio (not Visual Studio Code), using **MSVC** compiler.
* A loose group of toolchains called **MinGW**, which uses the same tools you'll commonly find on Linux.

We'll be installing the later, with the Clang compiler. *([Why MinGW rather than MSVC? And why Clang rather than GCC?](/why_mingw.md) and [Why not Visual Studio?](/why_not_visual_studio.md))*

More specifically, we'll be installing it using MSYS2. *([What is MSYS2? Why MSYS2?](/why_msys2.md) and [Why not the official Clang installer?](/why_not_official_clang_installer.md) and [Why not WSL?](/why_not_wsl.md))*

## Installing MSYS2

1. **Download the installer** from https://www.msys2.org/ and run it.

   (⚠ If you choose to install into a non-default location, make sure the path **doesn't contain any spaces or non-latin characters**. This tutorial assumes you installed to the default location `C:\msys64`.)

   If you run into any strange issues during installation (or later), try disabling your antivirus software or adding `C:\msys64` to its exceptions.

2. **Start MSYS2.**

   The "Start" menu will have several different shortcuts, for now you want `MSYS2 UCRT64` (read about the difference here).

   The window that opens is called a "terminal" or a "console". You can type commands in there. (I will explain more later.)

   **Make sure it says `UCRT64` in the purple text.** If it says something else, you used the wrong shortcut (see above). Close it and restart using the correct one.

   All commands below should be typed into the MSYS2 terminal.

3. **Update MSYS2.**

   Run `pacman -Syu` to install updates. (It's a good idea to do this from time to time.)

   It will eventually ask for confirmation: `:: Proceed with installation? [Y/n]`. Press <kbd>Enter</kbd> to continue.

   Sometimes you will get this: `:: To complete this update all MSYS2 processes including this terminal will be closed. Confirm to proceed [Y/n]`. Press <kbd>Enter</kbd> and the terminal will close. **If this happened, restart the MSYS2 terminal and run `pacman -Syu` again to finish the update.**

   When it's done updating, you'll see this again:
   ```sh
   username@computername UCRT64 ~
   $
   ```

* **Install toolchain.**

   Run `pacman -S mingw-w64-ucrt-x86_64-clang` to install Clang (which automatically brings in the rest of the toolchain).

   (⚠ Note that we install `mingw-w64-ucrt-x86_64-clang` and not just `clang`! The difference is explained [here](TODO_MSYS2_ENVS).)

   Now if you run `clang --version`, you should see something like this:
   ```
   clang version 18.1.6
   Target: x86_64-w64-windows-gnu
   Thread model: posix
   InstalledDir: C:/msys64_b/ucrt64/bin
   ```
   Which means Clang was installed correctly.

## Compiling and running a simple program

Now you can try compiling a simple program. See [How to compile and run programs in the terminal](/compiling_in_terminal_win.md).
