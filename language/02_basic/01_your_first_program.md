# Your first program

Let us start with the following simple program:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```

It should display `Hello, world!` on the screen when run. "Hello world" is
traditionally the first program learners write in most programming languages.

First, make sure you can actually run it, and after you do, then we'll discuss
how it works and what it means.

To run C++ programs you'll need to install some tools.

## The necessary tools

### The compiler

The compiler is a program that converts (or "compiles") the source you write
into a program that you can run - an "executable". You can send the resulting
executable to other people too, and they can run it without installing the
compiler, like you've been doing with various programs you've been using so far.

Not all languages require compilation. This, and the process of compilation will
be explained in more details in later chapters.

The popular C++ compilers are: Clang, GCC, and MSVC (in my order of preference,
but any of them will work fine).

### The IDE

Not strictly necessary, especially for a beginner. "IDE" stands for "integrated
development environment", where "integrated" means it *combines* together
various useful programming tools like a text editor, debugger and build system.

Some popular IDEs are: Visual Studio, CLion, XCode, QtCreator.

### How to install?

I have [**a separate tutorial**](/tooling/README.md) to guide you through the
process. That tutorial explains how to install and configure Visual Studio Code
with the Clang compiler (or GCC if you prefer). If you want to use something
else, install it on your own and come back here.

If you've already installed Visual Studio Code, I still recommend reading that
tutorial. Some people use VSC without really understand what's going on, and the
tutorial aims to fix that.

If you decide to use the tutorial, **only read the first few chapters**. As soon
as you're able to compile the simple program above, stop reading and leave it
for later. Continue learning C++ from here, and read that tutorial in parallel.

### Online compilers

There are websites that let you compile and run simple programs without
installing anything. They can sometimes be useful (to test small programs), but
are unsuitable for any serious work. I don't recommend using them for learning.
Install a proper compiler on your machine instead of using those.

## What this program means

Now you should be able to compile and run the simple program I've shown above. Here it is again:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```

If you've done everything correctly, this should print `Hello, world!` when run.

If you're typing it manually instead of copy-pasting, use the <kbd>Tab</kbd> key
to add the spaces before `std::cout` instead of hitting <kbd>Space</kbd>
multiple times. It's faster and also has other benefits.

### Strings

`"Hello, world!"` is a string - a sequence of characters - which we use to
represent text. Strings begin and end with `"`.

### Statements

The main part of the program is between those `{` and `}` (called "curly
brackets") which begin a block. Blocks contain **statements** that are executed
in order, top to bottom.

`std::cout << "Hello, world!\n";` is a statement that prints `Hello, world!`
when executed.

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

Statements can only appear inside of blocks. For example, the following is
illegal:
```cpp
#include <iostream>

int main()
{

}

std::cout << "Hello, world!\n";
```

### The `main` function

`int main() {...}` is the "`main` function".

Functions are sequences of statements that have names. The `main` function is
special. It is where execution begins when you run your program. Every C++
program must contain a function named `main`.

You will learn more about functions and what `int` and `()` are later.

As shown above, statements can only appear inside of blocks.

### The `#include` directive

The last unexplained part of the program is `#include <iostream>`. `#include` is
often called "the include directive", where "directive" is a word that means
"an order" or "a command".

Using certain features in a program requires using certain `#include`
directives at the top. For example, using `std::cout` to print requires using
`#include <iostream>`.

`iostream` is the name of a file that comes with your compiler. `#include`
copies and pastes the entire contents of the file you give it in place of itself
during compilation. The contents of `iostream` describe to the compiler what
`std::cout` is and enable you to use it.

There are many different **"header"** files that you can include and and your
compiler comes with a collection of header files called **the standard library**.
Things that come from the standard library have `std::` in their names to
indicate that.

People can make their own libraries (non-standard ones, called "third-party
libraries") to add functionality to C++ that it otherwise doesn't have, such as
the ability to work with graphics, which you'll need to make games.

### Meaning of words

`std` stands for "standard".

`cout` stands for "character output". It's what you use to print "characters" -
letters, numbers and symbols.

`iostream` stands for "input/output stream". `std::cout` is the output stream.

## More information

### Case sensitivity

C++ is said to be "case sensitive". This means `std::cout` and `sTd::COut` are
not the same thing, and trying to use the latter will not work.

### Whitespace and line breaks

Unlike some languages like Python, C++ is mostly insensitive to the number of
spaces you put between things and where you break lines.

For example, this program:
```cpp
#include <iostream>

int main()
{
    std::cout << "Hello, world!\n";
}
```
Does the same as this program:
```cpp
#include<iostream>
int main(){std::cout<<"Hello, world!\n";}
```
`#include` directives are an exception. They, as well as other directives - more
on that later, must always be on separate lines.

The space between `int` and `main` can't be removed, because then they would be
one word, but space between `cout` and `<<` can.

The space between `Hello,` and `world` can be removed, but that would change the
output text.

You can freely insert additional whitespace too:
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
But you shouldn't do this because it makes the code difficult to read.

We can't insert spaces between `::` and between `<<`, we can't split `cout` into
`co ut`, and so on.

### `\n` escape sequence

You probably already guessed that `std::cout << ...` means "print `...`", but
what does `\n` mean?

It's the "new line" symbol. For example, printing `"Hello,\nworld!\n"`, produces:
```
Hello,
world!
```
It's customary to add `\n` after the last line printed by your program.
You might run into some minor issues otherwise.

A `\` followed by some characters is called an **escape sequence**. Escape
sequences are used to insert various special characters into text.

To insert `\` itself, we use `\\`. For example, printing `"Hello\\world"` would
output `Hello\world`, whereas printing `"Hello\world"` is a compilation error
because the compiler tries to interpret `\w` as an escape sequence, which
doesn't exist.

Since strings begin and end with `"`, if we want to put a `"` inside of a
string, we have to use `\"`. Printing `"Hello\"world"` outputs `Hello"world`.

You can find the full list of escape sequences on [cppreference](https://en.cppreference.com/w/cpp/language/escape).

> ## Exercise
>
> As explained in the introduction, this tutorial includes exercises. Do them if you want. Regardless, practice on your own too. The exercise for this chapter is:
>
> Write a program that displays your favorite quote.
