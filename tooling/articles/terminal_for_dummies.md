# Terminals, shells, and PATH

## Terminal

A "terminal" (also called a "console") is a program that looks like this:

[![MSYS2 terminal](/tooling/images/terminal.png)](/tooling/images/terminal.png)

Terminals display text for other programs that are incapable of doing it themselves, such as:

* `pacman` (as on the screenshot above).
* Most compilers.
* Most simple C/C++ programs (that use `std::cin`, `std::cout`, etc).

A terminal doesn't decide *what* text to display, it just displays the text provided by another program. It also lets you type, and forwards your input to that program.

There are several different terminal programs, but there's very little difference between them. For example:

* mintty (MSYS2 uses it by default)

  ![mintty terminal](/tooling/images/terminal_mintty.png)

* Windows Terminal (default on Windows)

  ![windows terminal](/tooling/images/terminal_windows.png)

* VSCode's built-in terminal

  ![vscode terminal](/tooling/images/terminal_vscode.png)

They all do pretty much the same thing: draw text to the screen, and let you type things.

## Shells

If all terminals are similar, then why do some commands that work in MSYS2 terminal (such as your compiler: `clang++` or `g++`) not work in the Windows terminal or in VSC by default? (You might see errors such as `The term '...' is not recognized as ...`.)

Also, if you open those terminals yourself, you'll notice that the default text they display (called "prompt") is different from what I have on the screenshots above. You'll see this instead:

![vscode terminal with powershell](/tooling/images/terminal_vscode_powershell.png)

Notice `PS C:\...>` vs `user@computer CLANG64 ~` `$`. Why is that?

That is because on Windows different terminals use different *shells* by default, and those have different default settings.

### What is a shell?

A shell is a program that accepts commands from you (such as when you run a compiler: `clang++ prog.cpp` or `g++ prog.cpp`) and executes them (often by running other programs, in this case by running the Clang or GCC compilers respectively).

Shells also can't display text directly, and rely on terminals to do so. A shell and a terminal are usually used together. Starting a terminal usually defaults to starting some shell along with it.

In our case, different terminals are defaulting to different shells. As VSC helpfully shows on the screenshots above, the default shell in VSC (and on Windows in general) is called "Powershell", while the shell MSYS2 uses by default is "Bash".

Those different shells have different default settings on Windows, which is what affects which shell can or can't find your compiler.

It doesn't matter what terminal you use, what matters is your shell. So we either need to configure Powershell to do what we want, or configure VSC (or your terminal of choice) to use MSYS2 Bash instead of Powershell. Let's start with the former.

## PATH

When you run your compiler, `clang++` (or `g++`) in the MSYS2 shell, it runs a program named `clang++.exe` (or `g++.exe` respectively) located somewhere in the MSYS2 installation directory. If you followed the rest of this tutorial as is, it will be in `C:\msys64\clang64\bin`. [But if you didn't](./variations/determining_msys2_env.md), it could also be in `C:\msys64\ucrt64\bin` or `C:\msys64\mingw64\bin`.

[![clang exe](/tooling/images/clang_exe.png)](/tooling/images/clang_exe.png)

How does it know to look in this specific directory? Because of a shell setting called `PATH`.

## What is PATH?

PATH is a list of directories that a shell searches for a program when it needs to run it.

To see it, run `echo $PATH` in the MSYS2 shell and `echo $env:PATH` in Powershell in VSCode (the commands are different because these are two different shells, Bash and Powershell). You should see something like this:

[![default paths](/tooling/images/default_shell_paths.png)](/tooling/images/default_shell_paths.png)

You get two different lists of directories.

In MSYS2, the first one is `/clang64/bin` (`/` at the beginning refers to the MSYS2 installation directory, so this is actually `C:\msys64\clang64\bin`) (or, as I said above, you could see `/ucrt64/bin` or `/mingw64/bin` depending on how you configured MSYS2), which is what we need. In Powershell, this directory isn't there.

You can further confirm this by typing `which clang++` (or `which g++` if you use GCC) in the MSYS2 terminal, which will tell you where it thinks `clang++` is located. It should print `/clang64/bin/clang++`. (The equivialent command in Powershell is `gcm clang++`, but it should not find anything right now.)

As you might've guessed, you have to modify Powershell's PATH to include `C:\msys64\clang64\bin`. (Or a different directory, see above.)

## How to modify PATH?

Open the Start menu (the "Windows" button at the bottom of your screen), type `env` in the search box, choose `Edit the system environment variables` then click `Environment Variables...`.

You will see two lists of variables, with `Path` in both of them (see `1` and `2` on the image below). The top list (`User variables`) applies only to your user, while the bottom (`System variables`) applies to every user on this computer. **In this case, you want to modify `2`, in the `System variables`, and add your directory to the very beginning,** because it gives it the most priority ([otherwise you can run into issues](/tooling/articles/debugging_dll_issues.md)).

[![modifying path](/tooling/images/modifying_path.png)](/tooling/images/modifying_path.png)

If you've used other compilers before, or changed this setting, you might want to check `1` and `2` for traces of those other compilers, and remove those if you find them, just in case.

Make sure to click `OK` in all the windows above instead of just closing them, or your changes will be lost.

**Restart VSC (or your terminal of choice) after changing PATH.** A shell needs to be restarted to see the updated PATH value.

Now, in the VSC terminal try running your compiler again (e.g. `clang++ --version` or `g++ --version`), and it should work.

If you print the PATH now, you should see your directory in there, at the very beginning:

[![updated path](/tooling/images/updated_path.png)](/tooling/images/updated_path.png)

If it's not at the beginning, or is missing entirely, you didn't do the above steps correctly. Make sure you've added your directory to `2` instead of `1`, and that it's at the very beginning of `2`, and not elsewhere. And make sure you clicked

And if you run `gcm clang++` now (or `gcm g++` for GCC users), it should tell you that Clang (or GCC) is located at `C:\msys64\clang64\bin\clang++.exe` (or at whatever path you specified above).

---

You can stop reading here. The next section provides additional information about shells if you're curious.

## Additional information

### What shells are there?

The ones you can commonly see on Windows are:

* **Powershell**: As mentioned above, the default on Windows is Powershell, which you can recognize by its prompt `PS ...>`.
* **CMD**: This used to be the default on older Windows, and is still provided on the modern Windows. It has the same prompt but without `PS `, so `C:\foo\bar>` instead of `PS C:\foo\bar>`.

  CMD shares the PATH setting with Powershell.

* **Bash**: Normally used on Linux, but MSYS2 comes with a Windows port of it. The prompt you see in MSYS2 is a customized one.

### Accessing `pacman` in other terminals

`pacman` isn't located in `C:\msys64\clang64\bin`, it's in `C:\msys64\usr\bin`. So the above changes to the PATH are not enough to make `pacman` work.

You could add that directory to the path too, immediately **after** `C:\msys64\clang64\bin` (or `C:\msys64\ucrt64\bin` or whatever you're using).

But keep in mind that this will bring a lot of Linux-y tools with it, which could be confusing.

### How to use MSYS2 Bash in VSC?

Do this at your own risk, since the rest of the tutorial assumes Powershell. There shouldn't be any big differences though.

In VSC settings, search for `terminal integrated profiles windows`, click `Edit in settings.json`, and you'll see those:

[![VSC default integrated terminal profiles](/tooling/images/vsc_terminal_profiles_default.png)](/tooling/images/vsc_terminal_profiles_default.png)

You can overwrite the default value of this setting, this doesn't actually remove those defaults. Replace it with the following:
```json
"terminal.integrated.profiles.windows": {
    "MSYS2 CLANG64": {
        "path": "C:/msys64/usr/bin/bash.exe",
        "args": ["--login", "-i"], // `--login` is needed to automatically set certain environment variables on launch.
        "env": {
            "MSYSTEM": "CLANG64", // Select the desired MSYS2 subsystem.
            "CHERE_INVOKING": "1", // Automatically `cd` to the current directory on launch.
            "MSYS2_PATH_TYPE": "inherit", // Append the system PATH setting after the MSYS2 one.
        }
    },
    "MSYS2 UCRT64": {
        "path": "C:/msys64/usr/bin/bash.exe",
        "args": ["--login", "-i"],
        "env": {
            "MSYSTEM": "UCRT64",
            "CHERE_INVOKING": "1",
            "MSYS2_PATH_TYPE": "inherit",
        }
    },
    "MSYS2 MINGW64": {
        "path": "C:/msys64/usr/bin/bash.exe",
        "args": ["--login", "-i"],
        "env": {
            "MSYSTEM": "MINGW64",
            "CHERE_INVOKING": "1",
            "MSYS2_PATH_TYPE": "inherit",
        }
    },
    "MSYS2 MSYS": {
        "path": "C:/msys64/usr/bin/bash.exe",
        "args": ["--login", "-i"],
        "env": {
            "MSYSTEM": "MSYS",
            "CHERE_INVOKING": "1",
            "MSYS2_PATH_TYPE": "inherit",
        }
    },
},
```
I've added all the common MSYS2 variants in there. You don't have to paste all of them, and can keep only `MSYS2 CLANG64` (or the one you need).

This adds MSYS2 shells to this menu:

[![VSC terminal profile selector button](/tooling/images/vsc_terminal_profile_selector.png)](/tooling/images/vsc_terminal_profile_selector.png)

You can then make one of them the default, by changing the `Terminal Integrated Default Profile Windows`.
