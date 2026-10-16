# Function declarations

As briefly mentioned in a previous chapter, functions (like everything else you've seen so far) are sensitive to the order in which they are declared:

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

In this case it's not a big deal, because moving `SayHello` before `main` makes it work. But in some cases it's impossible to solve with reordering alone, for example:

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
But it doesn't compile, since `a` wants to call `b`, which is declared below `a`. And swapping `a` and `b` doesn't help, since then `b` would be unable to call `a` for the same reason.

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

`void b(int x);` is a **declaration** of the function `b`, but not its definition. Note that for the first time in this tutorial we have a declaration that is not also a definition at the same time.

`void b(int x) {...}` is both its declaration and definition. Most if not all definitions are also declarations, but not the other way around.

Declarations that are not definitions are sometimes informally called **forward declarations** (to distinguish from definitions). Specifically for functions, they are sometimes called function **prototypes**.

Function declarations that are not definitions are often informally called **prototypes**. Or people will sometimes say "a declaration" implying "a declaration that is not a definition", this has to be judged from context.

By providing a prototype you promise to the compiler that you will provide the definition later. Not doing so normally causes a compilation error, rather than a runtime one (not exactly a "compilation" one, but the nuance will be described later; the point is that it normally errors when creating the executable, not when running it).

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

It is only needed when you have 2+ functions calling each other in a circular manner. Which might sound like a very uncommon scenario, which is true.

There are other usecases for prototypes that are explained in future chapters.

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
This isn't much different from having a prototype outside of the function. It merely prevents other functions from using this prototype, forcing them to declare their own. I'm mentioning this for completeness.

Only function prototypes can be inside of other functions. Function definitions must be outside. C++ has a feature that can be used instead of local functions (called "lambdas") that will be explained in later chapters.

## Declarations of other things

It is possible to forward-declare (declare without defining) some other things too (other than functions). For example, global variables:

```cpp
extern int x; // A declaration, not a definition. Can't have an initializer.

int main()
{
    std::cout << x << '\n'; // 10
    x = 42;
    std::cout << x << '\n'; // 42
}

int x = 10; // Or `int x;`, then it'll be zeroed by default.
```

Here we're able to use `x` in `main` even though we defined it after main, because we've declared it before main.

In this case this isn't very useful, because you could just move the variable before the function, but this again becomes useful when multiple source files are involved.
