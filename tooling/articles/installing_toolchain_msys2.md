# Installing a toolchain on Windows

## Choosing a toolchain

There are two prominent toolchains on Windows:

* The one bundled with Visual Studio (not Visual Studio Code), using the **MSVC** compiler.
* A loose group of toolchains called **MinGW**, which uses the same tools you'd commonly find on Linux.

We'll be installing the later, with the Clang compiler. *([Why MinGW rather than MSVC? And why Clang rather than GCC?](/tooling/articles/choosing_compiler_and_more.md) and [Why not Visual Studio?](/tooling/articles/why_not_visual_studio.md))*

More specifically, we'll be installing it using MSYS2. *([Why is MSYS2? What's the difference between "MSYS2", "MinGW", etc?](/tooling/articles/why_msys2.md) and [Why not the official Clang installer?](/tooling/articles/why_not_official_clang_installer.md) and [Why not WSL?](/tooling/articles/why_not_wsl.md))*

## Alternative routes

<details><summary><b>I want GCC instead of Clang!</b></summary>

Are you sure?

This tutorial recommends Clang for a reason. In addition to being a good compiler in general, it gives you access to some powerful tools that'll help you catch bugs: [Address Sanitizer and UB Sanitizer](/tooling/articles/recommended_compiler_flags.md#flags-to-catch-errors). (GCC supports them on Linux and elsewhere, but not on Windows.)

Clang is made to be compatible with GCC, so if you're a newbie, you'll hardly notice any difference.

If you're sure: MSYS2 has several different versions of GCC available. A good one to choose by default is [MSYS2 UCRT64](/tooling/articles/variations/ucrt64_gcc.md). For the list of different GCC and Clang versions it provides, consult [this](/tooling/articles/variations//index.md) and [this](/tooling/articles/choosing_compiler_and_more.md).

</details>

<details><summary><b>I already have MSYS2 installed!</b></summary>

MSYS2 provides several different compilers to choose from. If you already installed one, it may or may not be different from the one this tutorial uses.

It's easier to follow this tutorial as is, especially if you don't know what you're doing. You can reinstall MSYS2 if you prefer to start from scratch, but you don't have to.

But if you want to reuse the existing compiler you have installed, consult [this](/tooling/articles/variations/determining_msys2_env.md).

</details>

<details><summary><b>I already have a compiler installed! (not in MSYS2)</b></summary>

If you're new, I recommend uninstalling it (or leaving it be), and following the tutorial as is. It'll be way easier, and there are specific reasons why I recommend this specific compiler distribution.

But if you insist on reusing it, which one do you have?

<details><summary><b>MSVC</b></summary>

Not covered by this tutorial. Also I have to admit, I don't see why someone would willingly use it for any reason other than it being the default choice in Visual Studio (not Code). [My rant on MSVC.](/tooling/articles/choosing_compiler_and_more.md#msvc-issues)

</details>

<details><summary><b>Clang from the official installer</b></summary>

Clang on Windows is not self-sufficient. It needs either a MinGW installation or a MSVC installation (the latter can come from Visual Studio (not Code)).

When you install Clang from MSYS2, it operates in MinGW-compatible mode by default, and MinGW is automatically installed with it. This is what this tutorial recommends.

When you install Clang from most other places, including from the official installer, it'll operate in MSVC-compatible mode by default, and will error if MSVC is not installed.

Both modes are viable. MinGW has a [slightly saner but less popular ABI](/tooling/articles/choosing_compiler_and_more.md#mingw-abi-vs-msvc-abi).

If you have MSVC installed, Clang should hopefully just work by default, but any non-trivial configuration of it in MSVC mode isn't covered by this tutorial.

</details>

<details><summary><b>MinGW / GCC / Clang from winlibs.com</b></summary>

Should mostly work fine, I guess. It should include most of the necessary tooling, but since there's no package manager, installing third-party libraries will have to be done manually.

</details>

<details><summary><b>MinGW / GCC from elsewhere</b></summary>

There are many bad outdated GCC distributions out there. If you don't know what you're doing, chances are, you got one of those. If you're curious, check your GCC version (run `gcc --version` to see it) and compare the number with the one at https://gcc.gnu.org/ (see the most recent version in `Supported Releases`).

Even if you got a decent up-to-date distribution, your next problem will be having to install all the necessary tooling manually, *and then configuring it* (e.g. the official version of CMake uses MSVC by default on Windows, and you'll have to manually tell it to use your compiler every time; whereas in MSYS2 it'll default to their compiler).

If that sounds ok, continue with the tutorial.

</details>

<details><summary><b>Something else / I don't know</b></summary>

Oh well. I suggest uninstalling whatever you have to avoid interference, then continue with the tutorial normally.

</details>

</details>

## Installing MSYS2

1. **Download the installer** from https://www.msys2.org/ and run it.

   There are two installers on that website, `msys2-x86_64-...` and `msys2-arm64-...`. You want the first one (unless your computer has an ARM processor, but those are rare, and you'd know if you had one).

   (⚠ If you choose to install into a non-default location, make sure the path **doesn't contain any spaces or non-latin characters**. This tutorial assumes you installed to the default location `C:\msys64`. I recommend not changing the location for simplicity.)

   Uncheck `Run MSYS2 now.` at the end of installation, or close the window that it opens.

   If you run into any strange issues during installation (or later), try disabling your antivirus software or adding `C:\msys64` to its exceptions.

2. **Start MSYS2.**

   The "Start" menu will have several different shortcuts, for now you want **`MSYS2 CLANG64`** (read about the difference [here](/tooling/articles/msys2_environments.md)).

   ![msys2 clang64 shortcut](/tooling/images/msys2_env_shortcuts.png)

   The window that opens is called a "terminal" or a "console". You can type commands in there. (I will explain more later.)

   **Make sure it says `CLANG64` in purple text.** If it says something else, you used the wrong shortcut (see above). Close it and restart using the correct one. (There are no lasting effects of typing commands in the wrong window. )

   All commands below should be typed into the MSYS2 terminal.

3. **Update MSYS2.**

   Run **`pacman -Syu`** to install updates. (It's a good idea to do this from time to time.)

   (⚠ If you want to paste commands from this tutorial, note that <kbd>Ctrl</kbd><kbd>V</kbd> doesn't work in MSYS2 terminal. Use <kbd>Shift</kbd><kbd>Insert</kbd> or right click → "Paste".)

   When you run the command above, it will eventually ask for confirmation: `:: Proceed with installation? [Y/n]`. Press <kbd>Enter</kbd> to continue.

   Sometimes you will get this: `:: To complete this update all MSYS2 processes including this terminal will be closed. Confirm to proceed [Y/n]`. Press <kbd>Enter</kbd> and the terminal will close. **If this happened, restart the MSYS2 terminal and run `pacman -Syu` again to finish the update.**

   When it's done updating, you'll see this again:
   ```sh
   username@computername CLANG64 ~
   $
   ```

* **Install toolchain.**

   Run **`pacman -S mingw-w64-clang-x86_64-clang`** to install Clang (which automatically brings in the rest of the toolchain).

   (⚠ Note that we install `mingw-w64-clang-x86_64-clang` and not just `clang`! The difference is explained [here](/tooling/articles/msys2_environments.md).)

   Now if you type `clang++ --version`, you should see something like this:
   ```
   clang version 18.1.8
   Target: x86_64-w64-windows-gnu
   Thread model: posix
   InstalledDir: C:/msys64/clang64/bin
   ```
   Which means Clang was installed correctly.

## Compiling and running a simple program

Now you can try compiling a simple program. See [How to compile and run programs in the terminal](/tooling/articles/compiling_in_terminal.md).
