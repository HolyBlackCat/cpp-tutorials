# Your first program

Let us start with the following simple program:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```

It should display `Hello, world!` on the screen when ran. This is a tradition for the first programs in most programming books and tutorials.

First, make sure you can actually run it, and after you do, then we'll discuss how it works and what it means.

To run C++ programs you'll need to install some tools...

## The necessary tools

### The compiler

The compiler is a program that converts (or "compiles") the source you write into a program that you can run (called an "executable"). You can send the resulting executable to other people too, and they can run it without installing the compiler (like you've been doing so far).

Not all languages require compilation. This, and the process of compilation will be explained in more details later.

The popular compilers are Clang, GCC, and MSVC (in my order of preference, but any of them will work fine).

### The IDE

Not strictly necessary, especially for a beginner. IDEs are basically text editors for programmers. "IDE" stands for "integrated development environment", where "integrated" means it *combines* together various useful programming tools.

Some popular IDEs are Visual Studio Code, Visual Studio (two unrelated programs despite the naming), CLion, XCode, QtCreator.

### How to install?

I have [**a separate tutorial**](/README.md) to guide you through the process. That tutorial explains how to install and configure Visual Studio Code with Clang or some other compiler. If you want to use something else, install it on your own and come back here.

I recommend not using an online compiler, and instead installing the proper compiler on your computer. An online compiler is convenient to test simple programs, but usually isn't suitable for any serious work.

If you already installed Visual Studio Code, I still recommend reading that tutorial. Some people use VSC without really understand what's going on, and my tutorial aims to fix that.

If you decide to use that tutorial, **read the first few chapters only**. As soon as you're able to compile the simple program above, stop reading that tutorial and leave it for later. Continue learning C++ from here, and read that tutorial in parallel.

## What this program means

Now you should be able to compile and run the simple program I've shown above. Here it is again:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```

This should print `Hello, world!` when ran, if you've done everything correctly.

If you're typing it manually instead of copy-pasting, use the <kbd>Tab</kbd> key to add the spaces before `std::cout` instead of hitting <kbd>Space</kbd> multiple times, this saves time.

### Comments

Lines starting with `//`

### Statements

The main part of the program is between those `{` and `}`. It contains zero or more **statements** that are executed in order, starting from the first one.

`std::cout << "Hello, world!\n";` is a statement, that prints `Hello, world!` when executed.

Let's try multiple statements:
```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
    std::cout << "This is a small test program.\n";
    std::cout << "It prints different things...\n";
}
```
This should now print:
```
Hello, world!
This is a small test program.
It prints different things...
```

Try running this program and experiment with adding more statements.

Statements can only appear inside of those `{...}`. For example, the following is illegal:
```cpp
#include <iostream>

int main()
{

}

std::cout << "Hello, world!\n";
```

### The `main` function

`int main() {...}` is the `main` function. It contains those statements (those inside of `{...}`).

Functions are sequences of statements that have names. The function named `main` is special, it is what's executed when you run the program. Every C++ program must contain a function named `main`.

You will learn more about functions later, and you'll learn what `int` and `()` are later.

As already shown above, statements can only appear inside of functions, not outside.

### The `#include` directive

The last unexplained part of the program is `#include <iostream>`. `#include` is often called "the include directive" (where "directive" is a word that means "an order" or "a command").

The short explanation is that using certain features requires adding certain `#include` directives at the top of the program. E.g using `std::cout` to print things requires adding `#include <iostream>`.

`iostream` is a name of a file that comes with your compiler. What `#include` does is pasting the entire contents of the file you give it in place of itself during compilation. The contents of `iostream` describe to the compiler what `std::cout` is and allow you to use it.

There are many different files that you can include (those files are called **headers**), and the collection of them that comes with your compiler is called **the standard library**. Things that come from the standard library normally have `std::` in their names to indicate that.

People can make their own libraries (non-standard ones, called "third-party libraries") to add features to C++ that it otherwise doesn't have (such as the ability to work with graphics, which you'll need if you want to make games).

### Meaning of words

`std` stands for "standard" and means "this thing comes from the standard library".

`cout` stands for "character output", because it's what you use to print characters (letters and other symbols).

`iostream` stands for "input/output stream". `std::cout` is often called "the output stream".

## More information

### Case sensitivity

C++ is said to be "case sensitive". This means `std::cout` and `sTd::COut` are not the same thing, and trying to use the latter will cause a compilation error.

### Whitespace and line breaks

C++ is mostly not sensitive to the amount of spaces you put between things, and to where you break the lines. (Unlike the Python language, for example.)

For example, this program:
```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```
Can be shortened to:
```cpp
#include<iostream>
int main(){std::cout<<"Hello, world!\n";}
```
`#include` directives are an exception from this rule, they always must be on separate lines. (As well as any other directives starting with `#` which you will learn later.)

The space between `int` and `main` can't be removed, because then they would be treated as one long word. But space between `cout` and `<<` can. In general, you have to keep at least one space if you have letters on both sides of it.

The space between `Hello,` and `world` can be removed, but that would change the text that is printed.

You can freely insert additional whitespace too, e.g.:
```cpp
  #  include  <iostream>

   int  main  (  )

 {

    std

      ::
                  cout

         <<

               "Hello, world!\n"
        ;

   }
```
But of course you shouldn't do this, because it looks ugly.

Here we can't insert spaces between `::` and between `<<`, and of course we can't split `cout` into `co ut`, and so on.

### `\n` escape sequence

You probably already guessed that `std::cout << ...` means "print `...`", but what does the `\n` mean?

It inserts the "new line" symbol. For example, if you do instead print `"Hello,\nworld!\n"`, you will see
```
Hello,
world!
```
It's customary to add `\n` after the last line printed by your program. You might run into some minor issues otherwise.

`\` followed by some characters is called an **escape sequence**. They are used to insert various special characters into strings.

Among others, `\\` is used to get the `\` character itself. E.g. printing `"Hello\\world"` would give you `Hello\world`, whereas printing `"Hello\world"` is a compilation error because the compiler tries to interpret `\w` as an escape sequence, and there's no such escape sequence.

Another escape sequence is `\"`, which is used to get the `"` character. `"Hello\"world"` results in `Hello"world`, whereas `"Hello"world"` is again a compilation error, because here the string ends at the middle `"`, and the `world"` that follows is junk that the compiler can't understand.

You can find the full list of escape sequences on [cppreference](https://en.cppreference.com/w/cpp/language/escape).
