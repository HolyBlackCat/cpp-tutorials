# How to compile and run programs in the terminal (on Windows)

First, create the source code file.

Open Notepad (or any other text editor such as Visual Studio Code; not Word though!).

Create a new file and paste following C++ code into it: (or the one from your book)
```cpp
#include <iostream>

int main()
{
    std::cout << "Hello!\n";
}
```

It will print `Hello!` when ran. Your C++ book will explain what this code means.

Press `File -> Save As` or <kbd>Ctrl</kbd><kbd>S</kbd> to save the file.

## Saving the file

The file name should end with `.cpp`, For example `prog.cpp`. (`.cpp` is called an "extension") (`.cpp` is for C++, use `.c` for C)

Save it to any directory. Make sure both the directory name and the file name don't contain any special characters, and preferably no spaces too. Some tools will choke on those. (Use English letters, digits, `_`, `.`.)

Open your directory:

[![Windows file explorer](/images/file_explorer.png)]((/images/file_explorer.png))

⚠ Make sure `View` -> `Show` -> `File name extensions` is **enabled**.

With it enabled, take a look at the file name again. It must end with `.cpp`, not e.g. `.cpp.txt`. Rename if needed.

## Opening the terminal

Copy the path to this directory by right-clicking the address bar and pressing `Copy Address`:

[![Windows file explorer](/images/file_explorer_address.png)]((/images/file_explorer_address.png))

(TODO rename to prog.cpp here)

Open MSYS2 terminal as described [here](/installing_toolchain_msys2.md#installing-msys2) by clicking `MSYS2 UCRT64` in the Start menu. (Make sure it says `UCRT64` in purple text. If not, you clicked the wrong thing.)

Make sure you know the basics of how to use a terminal. Consult [Terminal for Dummies](/terminal_for_dummies.md) if you don't.

Open your directory by typing `cd`, space, then pasting your path and pressing Enter. E.g. `cd C:\code`. (Consult [this](/terminal_for_dummies.md) if you have issues!)

## Compiling

Assuming you have Clang installed as explained [here](/installing_toolchain_msys2.md), running `clang++ prog.cpp` should compile your code. (This is for C++, for C use `clang prog.c`.)

A successful compilation will print nothing. If your code is wrong, errors will be printed. (TODO link to explanation for undefined reference to WinMain, write a new page for it)

A successful compilation will create a file called `a.exe`, this is a program you can run.

Run it in your terminal using `./a.exe` and you should see `Hello!` being printed.

## You did it! ❤️

This is enough to start learning C++.

Read a chapter or two in your C++ book and try a few simple programs.

Then [install an IDE](/installing_ide.md) for a better programming experience.

## More information

You can also run your executable by double-clicking `a.exe`, but the application will quickly close before you can see the results. (This is the correct behavior for programs that are supposed to be run in the terminal, but if you don't like it, Google for workarounds.)

You can tell the compiler to use a different executable name using `-o`, for example: `clang++ prog.cpp -o prog.exe` will name the program `prog.exe`. (The order doesn't matter, `clang++ -o prog.exe prog.cpp` works as well. But the executable name must immediately follow `-o`, e.g. `clang++ prog.cpp prog.exe -o` doesn't work. `-o prog` and `-o prog.exe` are equivalent.)

Remember that you can press "up" in the terminal to repeat the last command, instead of typing it every time.

You can use `clang++ prog.cpp -o prog.exe && ./prog.exe` to both compile and run the application as a single command.