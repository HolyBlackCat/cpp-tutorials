# Terminal for Dummies

## What is a "terminal" or a "console"?

A "terminal" or a "console" is a program that looks like this:

[![MSYS2 terminal](/images/terminal.png)](/images/terminal.png)

It lets you type in commands, and shows their results as text.

**Many programs (e.g. compilers) don't have their own graphical interface, and the terminal is the only way to interact with them.**

Words "terminal" and "console" are synonyms. There are several different terminal programs that do the similar thing, but there's very little difference between them.

## What is a "shell"?

A terminal is a simple program that merely draws letters to the screen.

It's often used with a "shell", which is what actually understands the commands you type, and runs the applications you tell it to run.

There are different shells, and while they are similar for basic usage, they have differences, so you must understand which one you're running.

The popular ones are:

* **CMD**, the legacy Windows shell
  ```
  Microsoft Windows [Version 10.0.22621.3737]
  (c) Microsoft Corporation. All rights reserved.

  C:\Users\Username>
  ```

* **PowerShell**, the new Windows shell
  ```
  Windows PowerShell
  Copyright (C) Microsoft Corporation. All rights reserved.

  Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

  PS C:\Users\Username>
  ```
  Notice the `PS` on the last line!

* **Bash**, a shell commonly used on Linux (and by MSYS2)
  ```
  Username@ComputerName UCRT64 ~
  $
  ```
  This is how MSYS2 Bash presents itself, but this text you see (a "prompt") can be customized and can look different (e.g. by default on Linux).

On Windows, you can start a terminal with Powershell or CMD from the Start menu (the latter is called `Command Prompt`).

## The current directory

As you can see, all shells print some directory name (`C:\Users\Username` or a cryptic `~` in the examples above) as a part of their "prompt" (the text they print when they ask you for a command).

It is the "current directory". Shell commands operate on files in the current directory by default.

### Paths in Bash on Windows

The paths Bash prints require an explanation. `~` stands for the "home directory". On Windows, you can run `cygpath -aw ~` to see what path it stands for (`C:\msys64\home\Username` for me).

In general, Bash on Windows uses Linux-style paths. `C:\foo\bar` is spelled as `/c/foo/bar`.

When the initial `/` is not followed by a drive letter, it refers to the MSYS2 installation directory. Run `cygpath -aw /` to confirm this (prints `C:\msys64` for me).

## Basic shell commands

* List files in the current directory: `ls` (or `dir` in [CMD](#what-is-a-shell)).

* Set current directory: `cd`.

  * E.g. to open `C:\msys64` type `cd C:\msys64` (in [Bash](#what-is-a-shell) can also use `cd /c/msys64`).

  * Move to parent directory: `cd ..`. E.g. if you're in `C:\msys64\ucrt64`, it will take you back to `C:\msys64`.

  * Move to a subdirectory: `cd DirectoryName`. E.g. if you're in `C:\msys64`, running `cd ucrt64` will take you to `C:\msys64\ucrt64`.

  When directory names contain spaces or special characters, you might need to wrap them in quotes. E.g. `cd "C:\Program Files"` works but `cd C:\Program Files` doesn't.

  CMD has a quirk where moving between drive letters requires `cd /d`. E.g. if you're in `C:\msys64`, moving to `Z:\Games` requires `cd /d Z:\Games`, whereas doing just `cd Z:\Games` will have no effect.

  Bash doesn't like `\` at the end of paths, e.g. `cd C:\msys64\` will require pressing Enter twice.

## Running executables

Now is a good time to read [How to compile and run programs in the terminal](/compiling_in_terminal_win.md) to know how to compile executables.

To run an executable from the current directory, use: (where `program.exe` is the executable name)

* [Bash](#what-is-a-shell): `./program.exe` or just `./program` (`.exe` is implied)
* [PowerShell](#what-is-a-shell): `.\program.exe` or `./program.exe` (or without `.exe`)
* [CMD](#what-is-a-shell): `program.exe` or `.\program.exe` (or without `.exe`)

As you can see, shells in general require `./` or `.\` to run executables from the current directory. In general, `.` refers to the current directory.

On Windows program names end with `.exe` by convention. On Linux they usually don't end with anything.

## PATH

When you run an executable without `.\` or `./`, instead of looking for it in the current directory, shells will look in a predefined list of directories. (CMD will look in both, see above.)

This list is called the `PATH`. It's one of the "environment variables", the system settings used by some programs.

You can view the current PATH (or any environment variable) using:

* [Bash](#what-is-a-shell): `echo $PATH`
* [Powershell](#what-is-a-shell): `echo $env:PATH`
* [CMD](#what-is-a-shell): `echo %PATH%`

When you run `clang++`, the shell in fact runs `clang++.exe` located in `C:\msys64\ucrt64\bin`, because that directory is in PATH!

### Changing PATH

PATH can be changed, you can add your own directories in there.

MSYS2 is a bit special in this regard, because it doesn't respect the system PATH setting, and uses its own. (TODO explain how to change this?)

For other shells on Windows, you can change PATH in the settings. Open the settings, type `env` in the search box, press `Edit the system environment variables` then `Environment Variables...`.

You will see two lists of variables, with `Path` in both of them. The top list (`User variables`) applies only to your user, while the bottom (`System variables`) applies to every user on this computer. You can edit either, but prefer to add your directories **to the beginning of the system-wide PATH** (the second one), because that gives them priority (TODO link to DLL issues).

You can add e.g. `C:\msys64\ucrt64\bin` in there to be able to run your compiler from any shell.
