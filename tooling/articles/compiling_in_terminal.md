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

This will `Hello!` when ran, but first you need to learn **how** to run it.

For now, press `File` → `Save As` or <kbd>Ctrl</kbd><kbd>S</kbd> to save the file.

**Note to VSC users:** Don't try to "run" the file using the VSC itself yet; for now we're just using it to create the file. Don't be alarmed by any red squiggles that VSC displays, we'll fix them later.

## Saving the file

The file name should end with `.cpp`, For example `prog.cpp`. (`.cpp` is called an "extension".) (`.cpp` is for C++, use `.c` for C.)

Save it to any directory. Make sure both the directory name and the file name don't contain any special characters, and preferably no spaces too. Some tools will choke on those. (Use English letters, digits, `_`, `.`.)

Open your directory:

[![Windows file explorer](/tooling/images/file_explorer.png)]((/tooling/images/file_explorer.png))

⚠ Make sure `View` → `Show` → `File name extensions` is **enabled**.

With it enabled, take a look at the file name again. It must end with `.cpp`, not e.g. `.cpp.txt`. Rename if needed.

## Opening the terminal

Copy the path to this directory by right-clicking the address bar and pressing `Copy Address`:

[![Windows file explorer](/tooling/images/file_explorer_address.png)]((/tooling/images/file_explorer_address.png))

Open MSYS2 terminal as described [here](/tooling/articles/installing_toolchain_msys2.md#installing-msys2) by clicking `MSYS2 CLANG64` in the Start menu. (Make sure it says `CLANG64` in purple text. If not, you clicked the wrong thing.)

The terminal has a concept of a "current directory", which you can change using the `cd` command. Type `cd "`, paste the path using <kbd>Shift</kbd><kbd>Insert</kbd> (because <kbd>Ctrl</kbd><kbd>V</kbd> doesn't work in MSYS2 terminal), type the closing `"`, then <kbd>Enter</kbd>.

[![terminal after cd](/tooling/images/terminal_after_cd.png)](/tooling/images/terminal_after_cd.png)

Notice the current directory being displayed in yellow text.

Notice that MSYS2 displays the paths in the Linux style, `/c/code/a` instead of `C:\code\a`.<br/>
`cd` works with both path styles, so you could do `cd /c/code/a`.

## Compiling

Assuming you have Clang installed as explained [here](/tooling/articles/installing_toolchain_msys2.md), running `clang++ prog.cpp` should compile your code. (This is for C++, for C use `clang prog.c`.)

A successful compilation will print nothing. If your code is wrong, errors will be printed. (One particular error you can get is `undefined reference to WinMain`, which means there's no `int main()` in your code. Often because you forgot to save the file and are compiling an empty file (hit <kbd>Ctrl</kbd><kbd>S</kbd>)).

A successful compilation will create a file called `a.exe`, this is a program you can run.

Run it in your terminal using `./a` (or `./a.exe`) and you should see `Hello!` being printed.

## You did it! ❤️

This is enough to start learning C++.

Read a chapter or two in your C++ book and try a few simple programs.

Then [**install an IDE**](/tooling/articles/installing_vsc.md) for a better programming experience.

## More information

You can also run your executable by double-clicking `a.exe`. If you see any errors (you probably will at this point), consult [Debugging DLL issues](/tooling/articles/debugging_dll_issues.md). Even if you fix this, our test application will quickly close after you run it, before you can see the results. This is the correct behavior for programs that are supposed to be run in the terminal (but if you don't like it, Google for workarounds.)

You can tell the compiler to use a different executable name using `-o`, for example: `clang++ prog.cpp -o prog` will name the program `prog.exe`. (The order doesn't matter, `clang++ -o prog prog.cpp` works as well. But the executable name must immediately follow `-o`, e.g. `clang++ prog.cpp prog -o` doesn't work. `-o prog` and `-o prog.exe` are equivalent.)

Remember that you can press "up" in the terminal to repeat the last command, instead of typing it every time.

You can use `clang++ prog.cpp -o prog && ./prog` to both compile and run the application as a single command.
