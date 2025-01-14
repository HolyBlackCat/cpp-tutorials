# Working in the VSCode terminal

VSCode has a built-in terminal that you can open via `View`→`Terminal`. Let's try doing a few simple things in it to make sure everything is configured correctly.

Don't install any extensions VSC suggests you yet, we'll get to that later.

### Running the compiler

Create a .cpp file in VSC and try to compile it in the VSC terminal, [like you did before in the MSYS2 terminal](/compiling_in_terminal.md).

Running the compiler (`clang++ prog.cpp -o prog.exe`) will likely result in an error:
```powershell
PS C:\code> clang++ prog.cpp -o prog.exe
clang++ : The term 'clang++' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is
correct and try again.
At line:1 char:1
+ clang++ prog.cpp -o prog.exe
+ ~~~~~~~
    + CategoryInfo          : ObjectNotFound: (clang++:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

This is to be expected. Read [Terminal for Dummies](/terminal_for_dummies.md) and do what it says.

After following all the steps, `clang++` should work.

If you see error `undefined reference to WinMain`, you didn't save your file and are compiling an empty file. Hit <kbd>Ctrl</kbd><kbd>S</kbd> and try again.

### Running your executables

After compiling your executable, try running it using `./prog.exe`.

Make sure it actually runs and `cout` successfully prints things.

If running an executable does nothing, read [Terminal for Dummies](/terminal_for_dummies.md) again. `C:\msys64\clang64\bin` should be first entry in the PATH, and it should be in the system-wide PATH setting rather than the user-specific one. Remember to restart VSC after changing the PATH. [Read this for more information.](/debugging_dll_issues.md)

---

As the next step, you might want to [**configure the code completion**](/configuring_code_completion.md).
