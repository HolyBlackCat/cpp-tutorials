# Terminal for Dummies

## What is a "terminal" or a "console"?

A "terminal" or a "console" is a program that looks like this:

[![MSYS2 terminal](/tooling/images/terminal.png)](/tooling/images/terminal.png)

It lets you type in commands, and shows their results as text.

**Many programs (including compilers) don't have their own graphical interface, and the terminal is the only way to interact with them.**

Words "terminal" and "console" are synonymous.

There are several different terminal programs, but there's very little difference between them. For example:

* mintty (MSYS2 uses it by default)

  ![mintty terminal](/tooling/images/terminal_mintty.png)

* Windows Terminal (default on Windows)

  ![windows terminal](/tooling/images/terminal_windows.png)

* VSCode built-in Terminal

  ![vscode terminal](/tooling/images/terminal_vscode.png)

As you can see, they all do pretty much the same thing: draw text to the screen, and let you type things.

## Different shells

But if you open the VSCode terminal yourself, by default you'll see something subtly different compared to the previous screenshot.

![vscode terminal with powershell](/tooling/images/terminal_vscode_powershell.png)

The text (called "prompt", because it prompts you for a command) is different: `PS C:\...>` vs `user@computer CLANG64 ~` `$`. Why is that?

You'll also find that it doesn't accept some commands that MSYS2 terminal does (try using `clang++` and you might get `The term 'clang++' is not recognized as ...`). Why?

That's because **by default it uses a different shell** ("powershell" instead of "bash", as VSCode helpfully displays above the terminal), that has different settings.

## What is a "shell"?

A shell is a program that accepts commands from you (such as the `clang++ prog.cpp` command you've used before) and executes them (often by running other programs, in this case by running the Clang compiler).

A terminal is a dumb program that displays text. A shell is what actually understands the commands you type.

**A shell and a terminal are usually used together**: the terminal provides the UI and draws text, while the shell interprets the user commands.

You can usually recognize a shell by its "prompt" (the text it prints when asking you for a command). **You have to know what shell you're using,** because as you noticed the difference can matter (`clang++` works in MSYS2 Bash but not in Powershell by default).

## Ok, so how do I use Clang in Powershell?

When you run `clang++` in MSYS2 shell, it runs a program `clang++.exe` located at `C:\msys64\clang64\bin`:

[![clang exe](/tooling/images/clang_exe.png)](/tooling/images/clang_exe.png)

How does it know to look in there? Because of a setting called PATH.

## What is PATH?

PATH is a list of directories that a shell searches for a program when it needs to run it.

To see it, run `echo $PATH` in the MSYS2 shell and `echo $env:PATH` in Powershell in VSCode (the commands are different because these are two different shells, Bash and Powershell). You should see something like this:

[![default paths](/tooling/images/default_shell_paths.png)](/tooling/images/default_shell_paths.png)

You get two different lists of directories.

In MSYS2, the first one is `/clang64/bin` (`/` at the beginning refers to the MSYS2 installation directory, so this is actually `C:\msys64\clang64\bin`), which is what we need. In Powershell, this directory isn't there.

## Modifying PATH

As you might've guessed, you have to modify Powershell's PATH to include `C:\msys64\clang64\bin`.

Open the Windows settings, type `env` in the search box, choose `Edit the system environment variables` then `Environment Variables...`.

You will see two lists of variables, with `Path` in both of them. The top list (`User variables`) applies only to your user, while the bottom (`System variables`) applies to every user on this computer. **In this case, you want to modify the `System variables`, and add your directory to the very beginning,** because it gives it the most priority ([otherwise you can run into issues](/tooling/articles/debugging_dll_issues.md)).

[![modifying path](/tooling/images/modifying_path.png)](/tooling/images/modifying_path.png)

**Restart VSC after changing PATH.** (A shell needs to be restarted to see the updated value.)

Now, in the VSC terminal try using `clang++` again (e.g. `clang++ --version`), and it should work.

If you print PATH now, you should see your directory in there, at the very beginning:

[![updated path](/tooling/images/updated_path.png)](/tooling/images/updated_path.png)
