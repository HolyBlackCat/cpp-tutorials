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

I recommend creating a directory for your programs directly in the `C:` drive.

Open your directory:

[![Windows file explorer](/tooling/images/file_explorer.png)](/tooling/images/file_explorer.png)

⚠ Make sure `View` → `Show` → `File name extensions` is **enabled**.

With it enabled, take a look at the file name again. It must end with `.cpp`, not e.g. `.cpp.txt`. Rename if needed.

## Trying to compile

Open MSYS2 terminal as described [here](/tooling/articles/installing_toolchain_msys2.md#installing-msys2) by clicking `MSYS2 CLANG64` in the Start menu. (Make sure it says `CLANG64` in purple text. If not, you clicked the wrong thing.)

Try typing `clang++ prog.cpp` to compile. (This is for C++, for C use `clang prog.c`.)

This will say `error: no such file or directory: 'prog.cpp'`, which is expected, because it doesn't know in what directory your file is. See the next section for how to fix this.

If it instead says `clang++: command not found`, it means you either didn't install Clang as was explained [here](/tooling/articles/installing_toolchain_msys2.md), or started the wrong MSYS2 terminal (must use `MSYS2 CLANG64`, see above).

## Changing the current directory

To fix this, first, copy the path to your directory by right-clicking the address bar and pressing `Copy Address`:

[![Windows file explorer](/tooling/images/file_explorer_address.png)](/tooling/images/file_explorer_address.png)

The terminal has a concept of a "current directory", which you can change with the `cd` command. Type `cd "`, paste the path using <kbd>Shift</kbd><kbd>Insert</kbd> (because <kbd>Ctrl</kbd><kbd>V</kbd> doesn't work in MSYS2 terminal), type the closing `"`, then <kbd>Enter</kbd>.

[![terminal after cd](/tooling/images/terminal_after_cd.png)](/tooling/images/terminal_after_cd.png)

Notice the current directory being displayed in yellow text.

Notice that MSYS2 displays the paths in the Linux style, `/c/code/a` instead of `C:\code\a`.<br/>
`cd` works with both path styles, so you could do `cd /c/code/a`.

## Compiling

Now try running `clang++ prog.cpp` again, it should work this time.

A successful compilation will print nothing. If your code is wrong, errors will be printed. (One particular error you can get is `undefined reference to WinMain`, which means there's no `int main()` in your code. Often because you forgot to save the file and are compiling an empty file (hit <kbd>Ctrl</kbd><kbd>S</kbd> in your text editor).

A successful compilation will create a file called `a.exe`, this is a program you can run.

Run it in your terminal using `./a` (or `./a.exe`) and you should see `Hello!` being printed.

## You did it! ❤️

This is enough to start learning C++.

Read a chapter or two in your C++ book and try a few simple programs.

Then [**install an IDE**](/tooling/articles/installing_vsc.md) for a better programming experience.

## More information

You can press "up" in the terminal to repeat the last command, instead of typing it every time.

You can combine the two commands together: `clang++ prog.cpp && ./a`. This will both compile and run your program.

You can tell the compiler to use a different executable name using `-o`, for example: `clang++ prog.cpp -o prog` will name the program `prog.exe`. (The order doesn't matter, `clang++ -o prog prog.cpp` works as well. But the executable name must immediately follow `-o`, e.g. `clang++ prog.cpp prog -o` doesn't work. `-o prog` and `-o prog.exe` are equivalent.)

You can run your executable by directly double-clicking `a.exe`. If you see any errors (you probably will at this point), you can consult [Debugging DLL issues](/tooling/articles/debugging_dll_issues.md). Even if you fix this, our test application will quickly close after you run it, before you can see the results. This is the correct behavior for programs that are supposed to be run in the terminal (but if you don't like it, Google for workarounds.)
