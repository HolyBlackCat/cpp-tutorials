# How to use a debugger in a terminal?

I assume you have some minimal C++ knowledge: already tried compiling a few simple programs, and know what "variables" are. If not, come back later.

## What is a debugger?

A debugger is a program that can run *your* program step by step, letting you observe the values of the variables and so on.

There are several popular debuggers: LLDB, GDB, and the Visual Studio debugger. They are commonly used with Clang, GCC, and MSVC [compilers](/installing_toolchain.md#what-is-a-compiler) respectively.

We will be using LLDB because we're already using Clang *([Why Clang?](/choosing_compiler_and_more.md))*, and because Visual Studio Code works well with it (there's a good plugin for it). But all of them should work fine.

LLDB and GDB are similar on the surface level, everything explained on this page should work with GDB as well.

## Debugging in a terminal

Go read [Terminal for Dummies](/terminal_for_dummies.md) if you haven't already.

Most debuggers (except the Visual Studio one, from what I know) can be used in a terminal. There are ways to use them with UI as well, but we'll be covering this later.

Most of the time you won't be debugging in a terminal. The goal of this tutorial is to give a minimal experience of doing so, before teaching more convenient methods.

## Installing LLDB

Install LLDB in MSYS2: `pacman -S mingw-w64-clang-x86_64-lldb`. *([Why not the official Clang installer?](/why_not_official_clang_installer.md))*

Confirm that it works by running `lldb --version`.

## Using LLDB

Let's start with the following test program.

I assume you already know the basics of C++. If you don't understand this program, go read your C++ book a bit more and come back.

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

Now, to execute it step by step, you need to create a "breakpoint", i.e. tell the debugger on what line of the code to pause.

Type **`b 5`** to put a breakpoint on line 5 (which should be `int x = 10;`).

Now if you type **`r`** again, you should see following:

[![lldb in action](/images/lldb_in_action.png)](/images/lldb_in_action.png)

The program is paused on line `5`. You can use **`n`** (short for `next`) to execute one line and go to the next one.

Print the values of variables using **`p`** (short for `print`). E.g. try `p x` and `p i`.

Or use **`fr v`** (short for `frame variable`) to print all variables.

Lastly, when you want to unpause your program, type **`c`** (short for `continue`).

Type **`quit`** to exit LLDB and return to the shell.

## Debugging functions

For this, you must understand what "functions" are. If you don't, read your C++ book more and come back later.

Consider this simple program with a function:
```cpp
#include <iostream>

int foo()
{
    int r = 10;
    r += 1;
    r += 2;
    r += 3;
    return r;
}

int main()
{
    int x = foo();
    std::cout << x << '\n';
}
```

Run this in the debugger. Place a breakpoint on line `int x = foo();`. Press **`n`** to go to the next line and notice that it jumps immediately to `std::cout`, without showing how the function executes. How do we debug the function?

Restart the program, but this time use **`s`** (short for `step`) instead of `n`. Notice that the debugger did *step* into the function:

[![lldb steps into a function](/images/lldb_steps_into_function.png)](/images/lldb_steps_into_function.png)

When there's no function to enter, `s` is equivalent to `n`.

Now typing `s` or `n` will debug `foo()` line by line.

If you forgot where `foo()` was called from, type **`bt`** (meaning "backtrace") to show the "call stack":

[![lldb backtrace](/images/lldb_backtrace.png)](/images/lldb_backtrace.png)

Here `frame #0` is the current line inside `foo()`, and `frame #1` is the line in `main()` that called `foo()`. Everything below that are the internals of the standard library that called your `main()`.

Typing e.g. **`f 1`** (short for "frame") will show the code at that location, so you don't need to look it up by the line number. **`f 0`** (or just **`f`**) will print the current line.



To return back to `main`, you either press `s` or `n` enough times, or type **`fin`** (or `finish`) to return to the calling function, back ot `int x = foo();`. But this time, typing `s` will jump to `std::cout << ...` (just like `n`) rather than entering `foo()`, because it already finished executing.



## Debugging real life errors

Consider this nonsensical program:
```cpp
#include <iostream>

int main()
{
    int x = 42;
    std::cout << x << '\n';
    delete &x;
    std::cout << x << '\n';
}
```
It's recommended if you understand what `delete` is. (If not, perhaps read your C++ book more and come back later.) But it's not strictly necessary.

The point is, this program compiles, but when you run it it fails with an error, because of the incorrect use of `delete`.

Compile and run it (without the debugger yet), observe that the first number is printed but the second isn't. **Notably it doesn't tell you what line caused the error,** and in more complex cases it might not be immediately obvious from the output.

Let's try to find the offending line with the debugger. Run this program in the debugger as explained above. Don't place any breakpoints yet, just type **`r`** to run. (Though pressing `n` a bunch of times is a valid debugging strategy.)

[![lldb shows a cryptic error](/images/lldb_cryptic_error.png)](/images/lldb_cryptic_error.png)

Since the error happens deep inside `delete`, it doesn't directly show you the offending line in your source file.

Type **`bt`** (meaning "backtrace") to see where `delete` was called from:

[![lldb cryptic backtrace](/images/lldb_backtrace_cryptic.png)](/images/lldb_backtrace_cryptic.png)

The ``frame #7 ... prog.exe`main at prog.cpp:7:5`` refers to the `delete &x` line in your program. Everything above it is the internals of `delete`. And as I already said above, everything below it is the internals of the standard library that called your `main()`.

**You always look for the first `frame` that belongs to your code, and that is the offending line.**

Lastly, type `f 7` (meaning "frame #7") to view that code:

[![lldb shows the offending frame](/images/lldb_offending_frame.png)](/images/lldb_offending_frame.png)

---

Next step: learn [how to debug in VSCode](/configuring_vsc_debugger.md).
