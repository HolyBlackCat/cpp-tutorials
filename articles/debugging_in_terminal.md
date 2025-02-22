# How to use a debugger in a terminal?

I assume you have some minimal C++ knowledge: already tried compiling a few simple programs, and know what "variables" are. If not, come back later.

## What is debugging?

"Debugging" refers to fixing a program when it doesn't work correctly, i.e. finding the issue in the code and fixing it. This is often done with the help of a *debugger*.

## What is a debugger?

A debugger is a program that assists you in debugging. It can run *your* program step by step, letting you observe the values of all variables and so on.

There are several popular debuggers: LLDB, GDB, and the Visual Studio debugger. They are commonly used with Clang, GCC, and MSVC [compilers](/articles/installing_toolchain.md#what-is-a-compiler) respectively.

We will be using LLDB *([Why LLDB?](/articles/why_lldb.md))*, but the other debuggers should work fine too.

LLDB and GDB are similar on the surface level, most things explained on this page should work with GDB as well.

## Debugging in a terminal

Go read [Terminal for Dummies](/articles/terminal_for_dummies.md) if you haven't already.

Most debuggers (except for the Visual Studio one, from what I know) can be used in a terminal. There are ways to use them with UI as well, but we'll be covering that later.

Most of the time you won't be debugging in a terminal (though sometimes this is the only option). The goal of this chapter is to give a minimal first-hand experience of doing so, before teaching more convenient methods.

## Installing LLDB

Install LLDB in MSYS2: `pacman -S mingw-w64-clang-x86_64-lldb`. *([Why not the official Clang installer?](/articles/why_not_official_clang_installer.md))*

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

First, you need to compile it with the **`-g`** flag, e.g. `clang++ prog.cpp -o prog -g`. This adds debugging information to it, allowing the debugger to interact with it in a meaningful way.

### Running a program in LLDB

Start LLDB on your program using **`lldb prog`**. You should see something like this:
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

Type **`b 5`** to put a breakpoint on line 5 (which should be `int x = 10;`). When more than one file is involved, you will need to a file name, e.g. `b main.cpp:5`.

Now if you type **`r`** again, you should see following:

[![lldb in action](/images/lldb_in_action.png)](/images/lldb_in_action.png)

The program is paused on line `5`. You can use **`n`** (short for `next`) to execute one line and go to the next line.

Print the values of variables using **`p`** (short for `print`). E.g. try `p x` and `p i`.

Or use **`fr v`** (short for `frame variable`) to print all variables.

Another useful command is "continue executing until a specific line", **`th u 10`** (shoft for `thread until 10`), which can be used to quickly break out of loops.

Lastly, when you want to unpause your program, type **`c`** (short for `continue`).

Type **`quit`** or **`exit`** to exit LLDB and return to the shell.

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

Run this in the debugger. Place a breakpoint on line `int x = foo();`. Press **`n`** to go to the next line and notice that it jumps immediately to `std::cout`, without showing you how the function executes. How do we debug the function?

Restart the program, but this time use **`s`** (short for `step`) instead of `n`. Notice that the debugger did *step into* the function:

[![lldb steps into a function](/images/lldb_steps_into_function.png)](/images/lldb_steps_into_function.png)

When there's no function to enter, `s` is equivalent to `n`.

Now typing `s` or `n` will debug `foo()` line by line.

If you forgot where `foo()` was called from, type **`bt`** (meaning "backtrace") to show the "call stack":

[![lldb backtrace](/images/lldb_backtrace.png)](/images/lldb_backtrace.png)

Here `frame #0` is the current line inside `foo()`, and `frame #1` is the line in `main()` that called `foo()`. Everything below that are the internals of the standard library that called your `main()`.

Typing e.g. **`f 1`** (short for "frame") will show the code at that location, so you don't need to look it up by the line number. **`f 0`** will print the current line. Just **`f`** alone will use the most recent number by default. You can also use **`up`** and **`down`** to switch between the frames.

To return back to `main`, you either press `s` or `n` enough times, or type **`fin`** (or `finish`) to return to the calling function, back at `int x = foo();`. But this time, typing `s` will jump to `std::cout << ...` (just like `n`) rather than entering `foo()`, because it has already finished executing.



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

## Debugging stuck programs

Another bit of knowledge is how to deal with "stuck" programs. Consider this code:

```cpp
#include <chrono>
#include <iostream>
#include <thread>

int main()
{
    int i = 0;
    while (true)
    {
        std::this_thread::sleep_for(std::chrono::seconds(1));
        std::cout << i++ << '\n';
    }
}
```

This endlessly prints numbers.

If you know what loop is being executed, you can place a breakpoint there. But if you don't, you might want to just pause the program, doesn't matter where.

This is achieved by pressing <kbd>**Ctrl**</kbd><kbd>**C**</kbd>, or alternatively by typing **`pro i`** (short for `process interrupt`). Then you can use `bt` as was explained above, to figure out where you are. To get meaningful results, you might need to first run **`t 1`** first to switch to the *main thread*. (Threads are parts of a program that run in parallel, at the same time as one another. In addition to your `main()` function which is run by the main thread, there might be other code executing in parallel, in other threads, either started by the standard library or by you.)

Try it.

---

Next step: learn [**How to debug in VSCode**](/articles/configuring_vsc_debugger.md).
