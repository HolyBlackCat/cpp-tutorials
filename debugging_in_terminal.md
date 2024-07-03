# How to use a debugger in a terminal?

I assume you have some minimal C++ knowledge: already tried compiling a few simple programs, and know what "variables" are. If not, come back later.

## What is a debugger?

A debugger is a program that can run *your* program step by step, letting you observe the values of the variables and so on.

There are several popular debuggers: LLDB, GDB, and the Visual Studio debugger. They are commonly used with Clang, GCC, and MSVC [compilers](/installing_toolchain.md#what-is-a-compiler) respectively.

We will be using LLDB because we're already using Clang *([Why Clang?](/choosing_compiler_and_more.md))*, and because Visual Studio Code ([which we'll be installing later](/installing_ide.md)) works great with it. But all of them should work fine.

## Debugging in a terminal

Go read [Terminal for Dummies](/terminal_for_dummies.md) if you haven't yet.

Most debuggers (except the Visual Studio one, from what I know) can be used in a terminal. There are ways to use them with UI as well, but we'll be covering this later.

**Usually you won't be debugging in a terminal. The goal of this tutorial is to give a minimal experience of doing so, before teaching more convenient methods.**

## Installing LLDB

Install LLDB in MSYS2: `pacman -S mingw-w64-ucrt-x86_64-lldb`. *([Why not the official Clang installer?](/why_not_official_clang_installer.md))*

Confirm that it works by running `lldb --version`.

## Using LLDB

Let's start with the following test program.

I assume you already know the basics of C++. If you don't understand this program read, go read your C++ book a bit more and come back.

```cpp
#include <iostream>

int main()
{
    int x = 10;
    for (int i = 0; i < 5; i++)
        x += i;
    std::cout << x << '\n';
}
```

### Compiling a program with debugging information

First, you need to compile it with the **`-g`** flag, e.g. `clang++ prog.cpp -o prog.exe -g`. This adds debugging information to it, allowing the debugger to interact with it in a meaningful way.

### Running a program in LLDB

Start LLDB on your program using **`lldb prog.exe`**. You should see something like this:
```
(lldb) target create "prog.exe"
(rrent executable set to 'C:\code\a\prog.exe' (x86_64).
(lldb)
```

Typing **`r`** (or `run`) will run your application. You'll see its window flash for a moment, and close immediately. You'll see the following message:
```
Process 9804 exited with status = 0 (0x00000000)
```
`0` means the program ran and finished successfully.

### Breakpoints

Now, to execute it step by step, you need to create a "breakpoint", i.e. tell the debugger on what line of code to pause.

Type **`b 5`** to put a breakpoint on line 5 (which should be `int x = 10;`).

Now if you type **`r`** again, you should see following:

[![lldb in action](/images/lldb_in_action.png)](/images/lldb_in_action.png)

The program is paused on line `5`. You can use **`n`** (short for `next`) to execute one line and go to the next one.

Print the values of variables using **`p`** (short for `print`). E.g. try `p x` and `p i`.

Lastly, when you want to unpause your program, type **`c`** (short for `continue`).

Type **`quit`** to exit LLDB.
