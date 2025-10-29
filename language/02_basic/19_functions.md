# Functions

As was mentioned at the very beginning of this tutorial, functions are named blocks of code that you can execute. Here is a simple example:

```cpp
#include <iostream>

void SayHello()
{
    std::cout << "Hello, world!\n";
}

int main()
{
    SayHello();
    SayHello();
    SayHello();
}
```

This prints `Hello, world!` three times.

First, notice that functions can only be declared outside of other functions. `void SayHello() {...}` is a **function declaration** (and simulaneously a **function definition**, like all other declarations you've seen so far, more on that later).

And so is `int main() {...}`, if you remember from earlier chapters. `main` is the special function that's called when the program starts, while all other functions have to be called manually.

Functions can call other functions, which themselves can call more functions, and so on. This is how large programs are composed.

## Function names

Typically, functions are named as `SayHello` or `sayHello`. `say_hello` is rarely used, primarily only in the standard library. And hopefully no one names their functions in all caps.

Function names have the same restrictions on them as the variable names (revisit the chapter on variables if needed).

The only addition is that function names starting with `_` underscore are reserved and shouldn't be used. That is in addition to the names containing `__` (two underscores in a row) anywhere, or starting with `_` followed by an uppercase letter (which doesn't really matter now, since the lone starting `_` is already banned for functions).

Again, revisit the chapter on variables if you forgot what it means for a name to be reserved.

> ## Exercise 1
>
> Write your own short program that uses functions.

## Variables in functions

If a variable is declared in a function, it will be destroyed when the function returns, which naturally follows the general rule that a variable is destroyed when leaving the `{...}` braces it was declared in:

```cpp
void foo()
{
    int x = 10;
}

int main()
{
    foo();

    std::cout << x << '\n'; // Error, `x` is not declared.
}
```
You'll get this error regardless of calling or not calling the function.

But also functions can't access variables from the caller (the function that called them):

```cpp
void foo()
{
    std::cout << x << '\n'; // Error, `x` is not declared.
}

int main()
{
    int x = 42;
    foo();
}
```

And this isn't just because `x` is declared lower in the source code. If you reorder those two functions, you'll get the same error (in addition to the error about `foo()` not being declared yet, how to work around this is explained later).

In general, what variable is visible where is fixed at compile-time, it doesn't depend on what happens at runtime (what functions do or don't get called).

## Global variables

There are different ways of sharing information with functions, the dumbest of which is **global variables**. That is, variables declared outside of functions. (As opposed to **local variables** declared inside of functions, which you've been using before. The local variables are also called **automatic variables**, for reasons that will become apparent later.)

```cpp
#include <iostream>

int x = 5;

void IncreaseX()
{
    x += 100;
}

int main()
{
    std::cout << "x = " << x << '\n'; // x = 5
    IncreaseX();
    std::cout << "x = " << x << '\n'; // x = 105
    IncreaseX();
    std::cout << "x = " << x << '\n'; // x = 205
}
```
There isn't much to say about those, how they work is fairly natural.

Global variables exist for (almost) the entire duration of the program, rather than having a limited lifetime (they are said to inhabit the **global scope**). More formally, they are said to have "static storage duration", as opposed to "automatic storage duration" of local (automatic) variables.

Global variables are initialized before the `main` runs, and are destroyed after it finishes running.

Global variables can't be uninitialized, they are zeroed by default (unlike local variables).

They have the same name requirements as functions (regarding reseved names, see above). In general, anything declared outside of a function follows those.

Global non-constant variables may seem convenient, but they should be avoided in most cases, because they make it hard to track what function modifies what variables.

We have more civilized and explicit ways of passing information between functions, which will be demonstrated in a moment.

But first...

### Examples of global variables

Do you know that you've been using global variables since the very first chapter? Can you guess where?

`std::cout` and `std::cin` are global variables. (Since what else could the be? Surely not functions.) But what types do they have? We'll leave this as a mystery for now.

## Passing parameters

Currently `IncreaseX()` always increases `x` by `100`. What if we want to increase it by different amounts?

We can introduce a **parameter**:

```cpp
#include <iostream>

int x = 5;

void IncreaseX(int value)
{
    x += value;
}

int main()
{
    std::cout << "x = " << x << '\n'; // x = 5
    IncreaseX(100);
    std::cout << "x = " << x << '\n'; // x = 105
    IncreaseX(2000);
    std::cout << "x = " << x << '\n'; // x = 2105
}
```
Here `int value` is a **parameter declaration**.

It is essentially local variable in the function:
```cpp
void IncreaseX()
{
    int value = /*magic*/;
    x += value;
}
```
Except that you can and must specify its value when calling the function. Notice `IncreaseX(100);` and `IncreaseX(2000);` in the example above.

Here `100` and `2000` are called **arguments** of the function. They are said to be **"passed"** to the function, or to that parameter.

Some people call parameters **"formal parameters"**, and arguments **"actual parameters"**. This is relatively rare though.

People often use the words "parameter" and "argument" loosely, saying one and meaning another. Do not be surprised by this, but try to use those words correctly.

### Multiple parameters

You can have more than one parameter, which then requires the matching number of arguments when calling the function. Not much to say about this, it's fairly straightforward.

```cpp
void IncreaseX(int value, int multiplier)
{
    x += value * multiplier;
}

int main()
{
    std::cout << "x = " << x << '\n'; // x = 5
    IncreaseX(10, 20);
    std::cout << "x = " << x << '\n'; // x = 205
}
```

### Passing variables to parameters

```cpp
#include <iostream>

int x = 5;

void IncreaseX(int value)
{
    x += value;
}

int main()
{
    std::cout << "x = " << x << '\n'; // x = 5
    int value = 100;
    IncreaseX(value);
    std::cout << "x = " << x << '\n'; // x = 105
}
```
Newbies sometimes get confused by this. The two `value` variables (one in `main` and another in `IncreaseX`) are completely unrelated, and just happen to have the same name.

There's nothing special about this. You could rename one of the variables and nothing would change:

```cpp
#include <iostream>

int x = 5;

void IncreaseX(int value)
{
    x += value;
}

int main()
{
    std::cout << "x = " << x << '\n'; // x = 5
    int y = 100;
    IncreaseX(y);
    std::cout << "x = " << x << '\n'; // x = 105
}
```

> ## Exercise 2
>
> Write two functions that print the area and the perimeter of a rectangle respectively.
>
> Write the `main` function so that you can input the width and height, then call those two functions.


## Modifying the parameters

Astute readers might've had an idea by now. If the global variables are so bad, what if we pass the variable to modify in a parameter?

```cpp
#include <iostream>

void Increase(int target, int value)
{
    target += value;
}

int main()
{
    int x = 5;
    std::cout << "x = " << x << '\n'; // x = 5
    IncreaseX(x, 100);
    std::cout << "x = " << x << '\n'; // x = 5 ?!
}
```
Sounds good in theory, but this doesn't work. `x` in `main` remains equal to `5`. But if you try to print `target` in `Increase` after changing it, you'll see that it's indeed `105`. What's going on?

You might have guessed already. `target` is a copy of `x`, so any changes to it don't affect the original. References to the rescue:
```cpp
void Increase(int &target, int value)
{
    target += value;
}
```
This now correctly increases the value of `x` in `main`.

Without `&`, the parameter variable is a copy of whatever variable/value you passed to it (initialized it with), just like any other variable. And making it a reference essentially makes it an "another name" for whatever variable (or something else) that you passed into it.

When a parameter is not a reference, it's said to be **passed by value**. When it is a reference, it's said to be **passed by reference**.

> ## Exercise 3
>
> Modify the functions from the previous exercise (that compute the area and perimeter) to write it to a reference parameter, instead of printing the result. Then print that in the `main` function after calling them.
>
> Typically functions that just perform computations shouldn't print anything, and shouldn't ask for input themselves. It's the job of the calling code.

## Returning values

Let's say we have a function that computes something:

```cpp
void ComputeStuff(int &out)
{
    out = 1 + 2 + 3; // Pretend there's a more complex computation here.
}

int main()
{
    int value;
    ComputeStuff(value);
    std::cout << value << '\n';
}
```
The part in `main` is fairly verbose, and kinda weird given that `ComputeStuff` doesn't care about the original value of `out`, and only uses `out` to output the result.

Would be nice to be able to write just `int value = ComputeStuff();`, and indeed we can do that!

```cpp
int ComputeStuff()
{
    int out = 1 + 2 + 3;
    return out;

    // Or just `return 1 + 2 + 3;` without introducing a variable.
}

int main()
{
    int value = ComputeStuff();
    std::cout << value << '\n';
}
```
Several things to note here. First, we now have `int ComputeStuff()` instead of `void ComputeStuff()`, as the function produces ("returns") a `int` instead of nothing (`void`). `int` here is the **return type** of the function.

Second, `ComputeStuff` now ends in `return ...;`. The value passed to `return` is the value that `main` receives in `int value = ComputeStuff();`.

> ## Exercise 4
>
> Modify your functions that compute area and perimiter to return the result, instead of writing it to a reference parameter.

## What about `int main` then?

Astute readers might be wondering: if the thing to the left of the function name is its return type, why have we been doing `int main` rather than `void main`? Or since we use `int`, why are we not returning anything from `main`?

The C++ standard says that `main` must have the `int` return type (doing otherwise should cause a compilation error on a properly configured compiler).

The `int` returned from `main` is used to indicate if the program has ran successfully (then you `return 0;`), or ended with an error (then you return some other number). Zero return code indicating success is universally agreed upon, but the specific meaning of non-zero codes isn't universal.

`main` defaults to `return 0;` if you don't `return` something else manually. Only `main` does this. **Forgetting to return from any non-void function (other than `main`) causes undefined behavior.** Typically it crashes the program (terminates it with an error at runtime).

#### Accessing the `main` exit code

Ok, so the return value of `main` indicates success vs failure. But how do you access this value when running the program?

This code is primarily used when you run your program from another program, not manually, to tell if it ran successfully or not.

If you run it manually, accessing the code isn't always simple. If you started the program just by double-clicking its executable, the exit code will typically be lost.

If you started the program in the [terminal](/tooling/articles/terminal_for_dummies.md#what-is-a-terminal-or-a-console), then the exit code can be accessed in a way that depends on your [shell](/tooling/articles/terminal_for_dummies.md#different-shells): if you're using Powershell, as is typical on Windows, then running `$lastexitcode` will print the exit code from the program you just ran. If you're using Bash (e.g. either in MSYS2 on Windows, or by default on most Linuxes) or something compatible, then `echo $?` will do the same thing.

## Returning conditionally

`return` doesn't have to be spelled at the end of the function. It can be anywhere, e.g. in an `if`. Reaching `return` immediately exits out of the current function. If this function is `main`, it exits the program.

You can have multiple `return`s per function too, but of course at most one can be reached at a time.

For example:
```cpp
#include <iostream>

bool IsSquare(int x)
{
    if (x < 0)
        return false;

    for (int i = 1;; i++)
    {
        int i_sq = i * i;
        if (x == i_sq)
            return true;
        if (x < i_sq)
            return false;
    }
}

int main()
{
    int x;
    std::cin >> x;
    if (IsSquare(x))
        std::cout << x << " is a square!\n";
    else
        std::cout << x << " is NOT a square!\n";
}
```
This program checks if a number is a square of some integer.

And here's an example of how `return` could be used in `main`:
```cpp
int main()
{
    int x;
    std::cout << "Enter a non-negative number: ";
    std::cin >> x;

    if (x < 0)
    {
        std::cout << "This number is negative!\n";
        return 1; // Non-zero exit code means an error.
    }

    if (IsSquare(x))
        std::cout << x << " is a square!\n";
    else
        std::cout << x << " is NOT a square!\n";
}
```

As a side note, you might be wondering how to check if the user entered something *other than* a number. That is achieved by using `std::cin` itself as a condition; it becomes false if something wrong is entered. E.g. `if (!std::cin || x < 0)` in the example above.

> ## Exercise 5
>
> Write a program that uses conditional `return` statements.

## Returning more than one value

While you can have any number of parameters, you can return at most one value. There's no real technical reason for this limitation.

Return a struct if you need more than one value.

## Returning from `void` functions

You can `return;` nothing in a `void` function to stop the function early.

For example:

```cpp
void foo(int x)
{
    if (x > 0)
        std::cout << x << '\n';
}
```

is equivalent to:

```cpp
void foo(int x)
{
    if (x <= 0)
        return;

    std::cout << x << '\n';
}
```

Trying to use `return;` (with no argument) in a non-`void` function is a compilation error, and similarly using `return ...;` (with an argument) in a `void` function is also *usually* a compilation error (with a rare exception that expressions of type `void` are allowed, such as, for example, calls to other functions returning `void`).

So:

```cpp
int a() {return 42;}
void b() {}

int c()
{
    return 42; // ok
    return; // error
    return a(); // ok
    return b(); // error, since `b` returns void
}

void d()
{
    return 42; // error
    return; // ok
    return a(); // error
    return b(); // ok
}
```

Here `return b();` isn't very useful, it's just a funny way of spelling `b(); return;`.

## Returning references

Just like with parameters, by default returning from a function creates a copy of the returned value.

But you can return references. For example:

```cpp
#include <iostream>

int num_positive = 0;
int num_negative = 0;

int &GetCounter(int number)
{
    if (number < 0)
        return num_negative;
    else
        return num_positive;
}

int main()
{
    while (true)
    {
        int x;
        std::cin >> x;
        if (x == 0)
            break;

        GetCounter(x) += 1;
    }

    std::cout << "Positive: " << num_positive << '\n';
    std::cout << "Negative: " << num_negative << '\n';
}
```
This program lets the user input numbers until they input 0, and counts positive and negative numbers in the input.

Notice `GetCounter(x) += 1;`. We can assign to `GetCounter(x)` because it returns a reference. If it didn't return a reference (in other words, if it "returned by value"), this would be a compilation error.

### Dangling references

I've already demonstrated dangling references in the chapter about references.

But functions give you new exciting ways of creating dangling references and the associated UB.

Let's say you return a reference to a local variable:

```cpp
#include <iostream>

int &foo()
{
    int arr[3];
    return arr[0];
}

int main()
{
    foo() = 42; // UB!
    std::cout << foo() << '\n'; // UB again!
}
```

This may appear to work, but this is in fact undefined behavior. One of the more treacherous kinds of UB, because it tends to not crash and to silently appear to work, until it no longer does.

Do you understand why this is UB? `int arr[3];` only lives until the end of this function call, but the returned reference to `arr[0]` outlives `arr`. This reference becomes **dangling**, meaning it no longer refers to a valid object.

This example can be further simplified:
```cpp
#include <iostream>

int &foo()
{
    int x;
    return x;
}

int main()
{
    foo() = 42; // UB!
    std::cout << foo() << '\n'; // UB again!
}
```
This has the exact same problem, but with one exception: this example no longer compiles in C++23 and newer. The compilers are now requires to provide basic protections against those mistakes. They are only *required* to in those basic cases, but still compilers typically try to warn about even the more complex cases.

Note that this specific example has stopped compiling in C++23 and newer, but you can fool the compiler and get the same UB using something like this:
```cpp
int &foo()
{
    std::vector<int> v = {1,2,3};
    return v[0];
}
```

Note that only **using** a dangling reference is UB. It merely existing doesn't cause UB by itself.

Note that if you return a reference to a reference parameter, that is fine:
```cpp
int &foo(int &x) {return x;}

int main()
{
    int y;
    foo(y) = 42;
    std::cout << y << '\n'; // 42
}
```
...since the returned reference ends up referencing `y` in `main` directly, not the parameter `x`. (As was explained before, references never refer to other references, trying to do so makes them refer to the target object of that reference.)

## Const references

If you [remember](./16_references_part_1.md#const-references), references can be const. Const references can serve as parameter types and return types types too.

### Const references as parameters

Those have a different purpose compared to non-const references parameters. They are primarily used to avoid expensive copying:


```cpp
void PrintVector(const std::vector<int> &v)
{
    for (int x : v)
        std::cout << x << '\n';
}

int main()
{
    std::vector<int> v(1000000); // Some large vector.
    // v[i] = ...
    PrintVector(v);
}
```

Here, `PrintVector()` takes the vector by a const reference, which means it's not copied when the function is called (copying a million `int`s just to print them would be unnecessarily slow), unlike if `PrintVector` had a non-reference parameter.

Passing by a const reference only makes sense for types that are expensive to copy. Something like `void PrintInt(const int &x)` doesn't usually make sense, something as small as an `int` should be passed by value.

Single numbers in general should be passed by value, and so are structs with just a few numbers in them. Vectors, strings, and structs of those should be passed by const references. (There are better ways to pass some of those, but those will be explained later. For now, a simple const reference is at least better that passing by value.)

### Const references as return types

Similarly, a const reference can be used as a return type if you need to avoid copying, and want to prevent the caller from modifying the result (which would be possible with a non-const reference).

```cpp
std::vector<int> a = {/*....*/};
std::vector<int> b = {/*....*/};

const std::vector<int> &GetVector(bool second)
{
    if (second)
        return b;
    else
        return a;
}

int main()
{
    std::cout << GetVector(true).size() << '\n'; // b.size()
    std::cout << GetVector(false)[42] << '\n'; // a[42]

    GetVector(false)[42] = 43; // Compilation error, because the reference is const.
}
```

Also notice that here you can make `a` and `b` `const`. But if `GetVector` returned a non-const reference, then trying to do so would cause a compilation error in `GetVector()` (can't bind non-const references to const variables).

## Arrays

Functions have a special relationship with arrays, for no particular reason other than C legacy. This only applies to arrays, and not to e.g. `std::vector`.

Arrays can't be returned from functions at all (specifying one as a return type is a compilation error).

When used as function parameters, arrays lose the size information:
```cpp
void foo(int arr[3]) // This `3` is ignored.
{
    // This is legal if the actual array passed into the function is large enough.
    // It doesn't matter that this is larger than `[3]` specified in the parameter list.
    std::cout << arr[4] << '\n';
}

int main()
{
    int blah[5] = {1,2,3,4,5}
    foo(blah); // Ok, despite the size mismatch in the function declaration.
}
```
For this reason, the array parameters are often spelled as `int arr[]` without the size, since it's ignored anyway. The only reason to specify the size there is as a hint to the programmer (if the function expects an array of a specific size).

There is no way for the function to know the true array size (if you remember `std::size(arr)` from earlier, it doesn't work on array parameters). But you can pass it manually in a separate parameter, e.g. `void foo(int arr[], int size)`.

You'll learn more about this in later chapters.
