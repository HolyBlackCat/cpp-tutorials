# How to debug DLL issues (on MinGW)?

This article explains how to fix following errors:

* `The code execution cannot proceed because __.dll was not found.`

* `procedure entry point __ could not be located in the dynamic link library __`

* Application simply doesn't start. (This will happen when launching it in a terminal or in an IDE. Double-click it in the file explorer* and you *should* see one of the above error messages.)

<sup>* Of course I mean the Windows "Files" application, not a tab in Visual Studio Code with the same name.</sup>

## A simple fix

If you're a newbie and you get this on your first program, the fix is to do everything inside MSYS2 terminal. Alternatively, [add `C:\msys64\clang64\bin` to PATH](/terminal_for_dummies.md#modifying-path). (Make sure you restarted your terminal and/or IDE after that, and that it's the first element of the system-wide PATH.)

But if you want to know more (or if you're using third-party libraries, or want to distribute your executables to other people), then continue reading.

## What is a DLL?

The executables you create are not standalone. When ran, they will immediately load a few so-called "DLL"s, and **fail to start if they can't**.

DLLs are files with the `.dll` extension. Some of them were installed with your compiler (look in `C:\msys64\clang64\bin`), some are preinstalled with Windows (look in `C:\Windows`, `C:\Windows\System32`, etc).

DLLs contain functions (compiled functions, not the source code) for your application to execute, and other things. By default your application loads DLLs to get access to some standard library functions.

"DLL" stands for a "dynamic link library" ("library" means "a collection of functions (and other things)", and "dynamic linking" refers to loading a library when the program runs, as opposed to baking it into the executable when creating it).

## Why do DLLs exist?

Why do some functions have to be in separate files, why can't everything be in a single `.exe`?

1. Several applications can use the same DLLs, saving disk space and possibly RAM (DLLs are also called "shared libraries", since they can be *shared* between applications).

2. A user can update the library themselves, without having to recompile the application (to apply bugfixes, etc).

## How does a DLL loading failure look like?

* **`The code execution cannot proceed because __.dll was not found.`** - DLL straight up wasn't found.

* **`procedure entry point __ could not be located in the dynamic link library __`** - a DLL with the correct name was found, but the contents are incompatible. Note that the DLL mentioned in the message isn't necessary the offending one, it can be a red herring.

* **Application simply doesn't start** - this will happen when launching it in a terminal or in an IDE. Double-click it in the explorer and you *should* see one of the above error messages.

## What DLLs my application uses?

There are several programs that can tell you this, one of them is NTLDD. Install it using `pacman -S mingw-w64-clang-x86_64-ntldd`.

**NOTE:** To get meaningful results from it, you must be able to run your application (it must not have [DLL issues explained above](#how-does-a-dll-loading-failure-look-like)). If it doesn't, either try running it in MSYS2 terminal (or [set PATH as explained above](#a-simple-fix)).

Run **`ntldd my_program.exe`** and you should see something like this:

```
libc++.dll => C:\msys64\clang64\bin\libc++.dll (0x00000258365c0000)
KERNEL32.dll => C:\Windows\SYSTEM32\KERNEL32.dll (0x0000025836730000)
```

You don't care about the libraries that come with the system (i.e. the first one in `C:\Windows`), only about those that come with the compiler (the second in `C:\msys64\clang64\bin`) or that you installed yourself.

**This isn't the full list though,** because those DLLs can load more DLLs themselves. To see all of them, run **`ntldd -R my_program.exe`** and you should see several dozens of them.

Most of those are irrelevant, because most of them are in `C:\Windows` (see above). You can use e.g. `grep` (a text filtering program) to filter the output. Run **`ntldd -R a.exe | grep clang64`** to only print the lines that mention `clang64`, and you should see something just this:
```
libc++.dll => C:\msys64\clang64\bin\libc++.dll (0x000002b7136c0000)
```
Even though in this case `-R` didn't add any relevant libraries, in general it might. Use it.

## Where does my application look for DLLs?

As you might've noticed from the NTLDD output ([see above](#what-dlls-my-application-uses)), your application loads DLLs from several different directories: `C:\msys64\clang64\bin`, `C:\Windows\System32`, etc.

Your application only knows the DLL file names it wants, not in which directories they are located.

It searches through several predefined directories. This is explained in the [Microsoft's manual](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order), but in short it searches those directories **in this order**:

* The directory where the `.exe` itself is located.

* `C:\Windows` and some of its subdirectories, like `C:\Windows\System32`.

* All directories in [PATH](/terminal_for_dummies.md#what-is-path), in the exact order they're listed (as explained in the link, there are two PATH settings, and it first searches system-wide PATH, then the user-specific one).

  (This is why when you add your compiler to PATH, I suggest adding it to the beginning of the system-wide PATH, to make sure all applications you compile load the DLLs from the compiler installation, and not from other applications you have added to PATH.)

## The solution

Therefore, the solution is to either copy the DLLs into the same directory as the .exe, or add their locations to PATH.

Copy the DLLs [that NTLDD prints](#what-dlls-my-application-uses) (ignoring those in `C:\Windows` and subdirectories, and only copying those installed by MSYS2 or by you manually) into the directory where you have your .exe.

While modifying PATH is ok for development purposes, you still must copy the DLLs when distributing your application to other people (and ship them with the application).

## Static linking

So-called "static linking" is an alternative solution. It refers to embedding some libraries into an .exe, meaning it no longer needs the respective DLLs. It can be enabled with the `-static` compiler flag. Confirm the result by running `ntldd -R my_program.exe` and observing that it loads absolutely nothing from `C:\msys64\clang64\bin`.

<!-- If you're using third-party libraries, refer to this (TODO third-party) -->

### Problems with static linking

While static linking is good for tiny portable applications (don't need to unzip a directory with a bunch of DLLs, or make an installer), it's usually not a good idea for serious applications, because:

* If a library is updated (e.g. bugs are fixed), your users can't apply the update themselves. (Can't download and replace a DLL, if the library is embedded into the executable.)

* Some libraries have licenses that penalize static linking. E.g any library licensed under [LGPL](https://en.wikipedia.org/wiki/GNU_Lesser_General_Public_License) (e.g. [OpenAL-soft](https://openal-soft.org/)), roughly speaking, forces you to either link dynamically or open-source your application.
