# Terminal for Dummies

## What is a "terminal" or a "console"?

A "terminal" or a "console" is a program that looks like this:

[![MSYS2 terminal](/images/terminal.png)](/images/terminal.png)

It lets you type in commands, and shows their results as text.

**Many programs (including compilers) don't have their own graphical interface, and the terminal is the only way to interact with them.**

Words "terminal" and "console" are synonymous. There are several different terminal programs, but there's very little difference between them (see below).

## What is a "shell"?

A terminal is a simple program that merely draws letters to the screen.

It's often used with a "shell", which is another program that actually understands the commands you type, and runs the applications you tell it to run.

There are different shells. While they are similar on the surface, they have differences, so **you must understand which shell you're currently using**.

## Terminals and shells

There are different terminals and different shells. The choice of shell can matter, but the choice of terminal usually doesn't.

It could be overwhelming, so take a look at the following picture: (you don't need to memorize the specifics)

[![terminals X shells](/images/terminals_x_shells.png)](/images/terminals_x_shells.png)

<sup>(1) is the default for MSYS2 (Bash in mintty)<br/>
(2) is the defualt on Windows (Powershell in Windows Terminal)<br/>
(3) was the defualt on old Windows (CMD in Conhost)</sup>

As you can see, there's very little difference between the rows (just the different font styles). In other words, the choice of terminal usually doesn't matter (usually). It just draws the text.

Also notice the different text in different columns. Different shells print different text when started (and accept some different commands too).

Those are the popular shells:

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
  Username@ComputerName CLANG64 ~
  $
  ```
  This is how MSYS2 Bash presents itself, but this text you see (a "prompt") can be customized and can look different (e.g. by default on Linux).

On Windows, you can start a terminal with Powershell or CMD from the Start menu (the latter is called `Command Prompt`).

## Terminal VS shell



## The current directory

As you can see, all shells print some directory name (`C:\Users\Username` or a cryptic `~` in the examples above) as a part of their "prompt" (the text they print when they ask you for a command).

It is the "current directory". Shell commands operate on files in the current directory by default.

### Paths in Bash on Windows

The paths Bash prints require an explanation. It prints Windows Paths in Linux style:

* `/c/foo/bar` means `C:\foo\bar`
* `~` means `C:\msys64\home\Username` (so `~/foo/bar` is `C:\msys64\home\Username\foo\bar`, and so on). `~` is called the "home" directory.
* `/` means `C:\msys64` (so e.g. `/ucrt64/bin` means `C:\msys64\ucrt64\bin`, and so on). `/` designates the MSYS2 installation directory.

You can use `cygpath -aw` to convert a Linux-style path to a Windows path, e.g. `cygpath -aw ~` should print `C:\msys64\home\Username`.

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

Now is a good time to read [How to compile and run programs in the terminal](/compiling_in_terminal.md) to know how to compile executables.

To run an executable from the current directory, use: (where `program.exe` is the executable name)

* [Bash](#what-is-a-shell): `./program.exe` or just `./program` (`.exe` is implied)
* [PowerShell](#what-is-a-shell): `.\program.exe` or `./program.exe` (or without `.exe`)
* [CMD](#what-is-a-shell): `program.exe` or `.\program.exe` (or without `.exe`)

As you can see, shells in general require `./` or `.\` to run executables from the current directory. In general, `.` refers to the current directory.

On Windows program names end with `.exe` by convention. On Linux they usually don't end with anything.

If you're enable to run executables you created, refer to [this](/debugging_dll_issues.md).

## Repeating previous commands

You can press Up arrow (and then Enter) to repeat the previous command. You can press Up or Down several times to switch between previous commands.

## PATH

When you run an executable without `.\` or `./`, instead of looking for it in the current directory, shells will look in a predefined list of directories. (CMD will look in both, see above.)

This list is called the `PATH`. It's one of the "environment variables", the system settings used by some programs.

You can view the current PATH (or any environment variable) using:

* [Bash](#what-is-a-shell): `echo $PATH`
* [Powershell](#what-is-a-shell): `echo $env:PATH`
* [CMD](#what-is-a-shell): `echo %PATH%`

When you run `clang++`, the shell in fact runs `clang++.exe` located in `C:\msys64\ucrt64\bin`, because that directory is in MSYS2's PATH!

### Changing PATH

PATH can be changed, you can add your own directories in there.

E.g. it's often desirable to add `C:\msys64\ucrt64\bin` in there to be able to run your compiler from any shell, not just from MSYS2 (which has it in PATH by default; MSYS2 is a bit special in this regard, because it doesn't respect the system PATH setting, and uses its own.) (TODO explain `-full-path`)

For other shells on Windows, you can change PATH in the settings. Open the settings, type `env` in the search box, press `Edit the system environment variables` then `Environment Variables...`.

You will see two lists of variables, with `Path` in both of them. The top list (`User variables`) applies only to your user, while the bottom (`System variables`) applies to every user on this computer. You can modify either, but if you're adding `C:\msys64\ucrt64\bin`, **it should be the beginning of the system-wide PATH** (the second one), because it gives it the most priority. [Otherwise you can run into issues.](/debugging_dll_issues.md)

**NOTE:** You must restart your terminal after changing PATH for it to take effect. (If you're running things in VSCode or any other IDE, restart it as well.)

### Changing PATH temporarily

You can make temporary changes to PATH that only affect the current shell and last until it's closed.

* [Bash](#what-is-a-shell): `export PATH="/c/foo/bar:$PATH"`
* [Powershell](#what-is-a-shell): `$env:PATH = "C:\foo\bar;$env:PATH"`
* [CMD](#what-is-a-shell): `set "PATH=C:\foo\bar;%PATH%"`

This prepends `C:\foo\bar` to PATH. [Print it](#path) before and after to confirm the change.

## Running several commands in a row

You often want to run several commands in a row. E.g. run the compiler first, and if it succeeds, run the executed command.

You can do it like this:
```sh
clang prog.cpp -o prog.exe && ./prog.exe
```
Old [PowerShell](#what-is-a-shell) doesn't understand `&&`. Either update it or do this:
```powershell
clang prog.cpp -o prog.exe ; if ($?) { ./prog.exe }
```
