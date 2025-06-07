# Function declarations

As briefly mentioned in the previous chapter, functions (like everything else you've seen so far) are sensitive to the order in which they are declared:

```cpp
#include <iostream>

int main()
{
    SayHello(); // Error, `SayHello` wasn't declared (yet).
}

void SayHello()
{
    std::cout << "Hello, world!\n";
}
```

In this case it's not an issue, because defining `SayHello` before `main` makes it work. But in some cases it's impossible to solve with reordering alone, for example:

```cpp
#include <iostream>

void a(int x)
{
    if (x == 0)
        return;

    std::cout << "a(" << x << ")\n";
    b(x - 1); // error, `b` is not declared
}

void b(int x)
{
    if (x == 0)
        return;

    std::cout << "b(" << x << ")\n";
    a(x - 1);
}

int main()
{
    a(4);
}
```

If this compiled, it would print:
```
a(4)
b(3)
a(2)
b(1)
```
But it doesn't compile, since `a` wants to call `b`, which is declared below `a`. And swapping `a` and `b` doesn't help, as then `b` would be unable to call `a` for the same reason.

The solution to this is...

## Declarations

This can only be solved using **function declarations**:

```cpp
#include <iostream>

void b(int x); // <-- This is a declaration.

void a(int x)
{
    if (x == 0)
        return;

    std::cout << "a(" << x << ")\n";
    b(x - 1);
}

void b(int x)
{
    if (x == 0)
        return;

    std::cout << "b(" << x << ")\n";
    a(x - 1);
}

int main()
{
    a(4);
}
```

`void b(int x);` is a declaration of the function `b`, but not its definition.

`void b(int x) {...}` is both its declaration and definition. Most if not all definitions are also declarations, but not the other way around.

Function declarations that are not definitions are often informally called **prototypes**. Or people will sometimes say "a declaration" implying "a declaration that is not a definition", this has to be judged from context.

By providing a prototype you promise to the compiler that you will provide the definition later. Not doing so is causes a compilation error (or not exactly a "compilation" one, but the nuance will be described later; the point is that it typically errors when creating the executable, not when running it).

## Functions calling themselves

If a function calls itself, no prototype is needed:

```cpp
#include <iostream>

void a(int x)
{
    if (x == 0)
        return;

    std::cout << "a(" << x << ")\n";
    a(x - 1);
}

int main()
{
    a(4);
}
```

It is only needed when you have 2+ functions calling each other in a circular fashion. Which might sound like a very uncommon scenario, which is true.

A much more common usecase for prototypes is separating (large) programs into multiple `.cpp` files (where one file would define a function, and another would declare it to be able to call it). How to do this is described in later chapters.

## Local prototypes

Going back to the first example of this chapter, another possible solution is making the prototype local in `a`:
```cpp
void a(int x)
{
    void b(int x);

    if (x == 0)
        return;

    std::cout << "a(" << x << ")\n";
    b(x - 1);
}
```
This doesn't have much use, it merely prevents other functions from using this prototype, and forcing them to declare their own.

This is mentioned here for completeness, and is almost never used.

Only function prototypes can be inside other functions. Function definitions must be outside. C++ has a feature to replace local functions (lambdas), which will be explained in later chapters.

## Declarations of other things

There are ways to declare things other than functions
