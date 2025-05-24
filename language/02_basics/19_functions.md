#### A word of caution

Starting from this point, the amount of information increases a lot.

As usual, don't feel the need to memorize everything. Practice by write small programs that use every new concept that you read about. Then feel free to return to this and previous chapters if you forget things and need to look them up.

# Functions

As was mentioned in the very beginning of this tutorial, functions are named blocks of code that you can execute. Here is a simple example:

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

## Function names

Typically, functions are named as `SayHello` or `sayHello`. `say_hello` is rarely used, primarily only in the standard library. Hopefully no one names their functions in all caps.

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

In general, what variable is visible where is fixed at compile-time, it doesn't depend on what happens at runtime (what functions get or don't get called).

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

Global variables exist for the entire duration of the program, rather than having a limited lifetime (they are said to inhabit the **global scope**). More formally, they are said to have "static storage duration", as opposed to "automatic storage duration" of local (automatic) variables.

Global variables are initialized before `main` runs, and are destroyed after it runs.

They have the same name requirements as functions (regarding reseved names, see above). In general, anything declared outside of a function has those.

Global mutable (that is, non-constant) variables are rightly considered a bad practice and should be avoided in most cases. (Because they make it hard to track what function modifies what variables.)

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

It's essentially local variable in the function:
```cpp
void IncreaseX()
{
    int value = /*magic*/;
    x += value;
}
```
Except that you can and must specify its value when calling the function. Notice `IncreaseX(100);` and `IncreaseX(2000);` in the example above.

Here `100` and `2000` are called **arguments** of the function. They are said to be **"passed"** to the function or to that parameter.

Some people call parameters **"formal parameters"**, and arguments **"actual parameters"**. This is relatively rare though.

People often use the words "parameter" and "argument" loosely, saying one and meaning another. Do not be surprised by this, but try to use those words correctly.

### Multiple parameters

You can have more than one parameter, which then requires the same number of arguments when calling the function. Not much to say about this, it's fairly straightforward.

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

There's nothing special about this. You could rename one of the variables and nothing would change.

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
Sounds good in theory, but this doesn't work. `x` in `main` remains as `5`. But if you try to print `target` in `Increase` after changing it, you'll see that it's indeed `105`. What's going on?

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
The part in `main` is fairly verbose, and kinda weird given that `ComputeStuff` doesn't care about the original value of `out`, and only uses it to output the result.

Would be nice to be able to write just `int value = ComputeStuff();`, and indeed we *can* do that!

```cpp
int ComputeStuff()
{
    int out = 1 + 2 + 3;
    return out;

    // Or just `return 1 + 2 + 3;`.
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

Astute readers might be wondering: if the thing to the left of the function name is its return type, why have we been doing `int main` rather than `void main`? Or if we use `int`, why are we not returning anything from `main`?

The C++ standard says that `main` must have the `int` return type. This `int` is used to indicate if the program has ran successfully (then you `return 0;`), or ended with an error (then you return some other number). Zero return code indicating success is universally agreed upon, but specific meaning of non-zero codes isn't universal.

`main` defaults to `return 0;` if you don't `return` something else manually. Only `main` does this. **Forgetting to return from any non-void function (except `main`) causes undefined behavior.** Typically it crashes the program (terminates it with an error at runtime).

#### Accessing the `main` exit code

Ok, so the return value of `main` indicates success vs failure. But how do you access this value?

This code is primarily used when you run your program from another program, not manually, to tell if it ran successfully or not.

If you run it manually, accessing the code isn't always simple. If you started the program just by double-clicking its executable, the exit code will typically be lost.

If you started the program in the [terminal](/tooling/articles/terminal_for_dummies.md#what-is-a-terminal-or-a-console), then the exit code can be accessed in a way that depends on your [shell](/tooling/articles/terminal_for_dummies.md#different-shells): if you're using Powershell, as is typical on Windows, then running `$lastexitcode` will print the exit code from the program you just ran. If you're using Bash (e.g. either in MSYS2 on Windows, or by default on most Linuxes) or something compatible, then `echo $?` will have the same effect.

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
        std::cout << "This is not a non-negative number!\n";
        return 1; // Non-zero exit code means an error.
    }

    if (IsSquare(x))
        std::cout << x << " is a square!\n";
    else
        std::cout << x << " is NOT a square!\n";
}
```

As a side note, you might be wondering how to check if the user entered something *other than* a number. That is achieved by using `std::cin` itself as a condition; it becomes false if something wrong is entered. E.g. `if (!std::cin || x < 0)` in the example above.

> ## Exercise 7
>
> Write a program that uses conditional `return` statements.

## Returning more than one value

While you can have any number of parameters, you can return at most one value. There's no real technical reason for this limitation.

Return a struct if you need more than one value.

## Passing and returning arrays

Arrays are a bit of a special case.

They can't be returned functions (at least not directly). But putting it in a struct allows you to return the whole struct.

They can serve as parameters, but when used as a parameter, the array size is ignored. E.g. you can do `void foo(int array[10])`, and then pass `int arr[5]` to it, and the array size mismatch isn't an error.

In fact, you can omit the array size in the parameter entirely: `void foo(int array[])`. This is equivalent to the above.

In all those cases, if you want to accept arrays of different sizes, you must manually pass the array size in a separate parameter. You may remember `std::size` from previous chapters, which can normally be used to determine the size of an array, but no, it doesn't work on array parameters.

This will be explained in more details later, including the exact nature of those fake array parameters.

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

Notice that we can assign to `GetCounter(x)` because it returns a reference. If it didn't return a reference (in other words, if it "returned by value"), we couldn't assign to that.

When a function returns by value, the result of the function call is an rvalue. When it returns by reference, it's an lvalue. That's why you can or can't assign to those.

### Dangling references

I've already demonstrated dangling references in the chapter about references.

But functions give you new exciting ways to create dangling references and the associated UB.

Let's say you return a reference to a local variable:

```cpp
#include <iostream>

int &foo()
{
    int x;
    return x;
}

int main()
{
    foo() = 42;
    std::cout << foo() << '\n';
}
```
This may appear to work, but this is in fact undefined behavior. One of the more treacherous kinds of UB, because it tends to not crash and to silently appear to work, until it no longer does.

Do you understand why this is UB? `int x;` only lives until the end of the function call, but the returned reference to `x` outlives `x`. This reference becomes **dangling**, meaning it no longer refers to a valid object.

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





explain the declarations (probably between parameters and return); also explain returning const (after the rest about return)
