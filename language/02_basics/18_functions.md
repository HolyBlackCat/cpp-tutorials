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

This is the same issue as we had with ranged-for loop a few chapters ago (revisit that chapter if you forgot). And it has the same solution, **references**:
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

## More about references

Okay, so now what happens if we do `Increase(5, 100);` instead of passing a variable to the first parameter?

Same thing as what happens if you try this with a non-parameter reference variable:
```cpp
int main()
{
    int &target = 5;
}
```
That being a compilation error.

The reason for this should be intuitively obvious, at least for functions. If you have `void Increase(int &target, int value)`, this implies that you want to access value of the first argument in the caller after calling the function (or you wouldn't have made it a reference), so the compiler prevents you from using the function "incorrectly".

In other words, if `Increase(5, 100)` compiled and updated the `5` to `105`, how would you read its new value if it's unnamed (isn't a named variable)?

But...

## Value categories

...it is interesting to explore what happens here on a deeper level.

Why *exactly* is `x` (given `int x;`) a valid initializer for a reference variable (or a valid argument for a reference parameter), but `5` isn't?

You'll also notice that
```cpp
int x;
std::cin >> x;
```
...is valid, while `std::cin >> 5;` isn't.

Similarly, `x = 10;` is valid but `5 = 10;` isn't.

Clearly there's a more general rule at play here.

You might assume that those things are only legal for variables, but that's clearly not it, because array elements are not variables (only the entire arrays are), but can be `std::cin >>`ed into, assigned, etc. And the same is true for the struct members (`boss.health`), and those aren't really variables either (only the entire struct variable is a variable).

### lvalues and rvalues

Variables (like `x`) are **lvalues**. If your variable is an array or a struct, then its elements/members are also lvalues.

Numeric literals (like `5`) are **rvalues**. Results of most operators are rvalues, such as `10 + 20`. Results of most casts are rvalues too, such as `double(42)`.

"lvalue" and "rvalue" are **value categories**.

This is the missing component. `5 = 10;` doesn't compile because `5` is an rvalue, and `std::cin >> 5;` doesn't compile for the same reason. `int x;` `x = 5;` compiles because `x` is an lvalue, and so on.

"lvalue" originally stood for "left value", meaning it can be the lhs (left operand) of `=` assignment. But it can be the rhs too, of course.

"rvalue" stood for "right value", meaning it can only appear on the right side of an assignment.

Not all lvalues are assignable, so "can be the lhs of assignment" isn't a proper definition of lvalue. For example, `const` variables are still lvalues, but can't be assigned. Arrays can't be assigned either, despite being lvalues. Same for structs with const members, and those const members themselves.

A const variable can't be passed to an `int &target` parameter, or used with `std::cin >>`, so being an lvalue isn't the only requirement for those. Additionally they require the lack of constness.

### Summary

Lvalues (lvalue expressions) usually refer to things that will live for some more time. Whereas rvalues (rvalue expressions) usually refer to things that are about to be destroyed. Variables live for some time. Literals such as `5` don't exist outside of the expression they're used in.

There's more to this classification than just "lvalues" and "rvalues". Rvalues are further split into two categories, but they aren't very important, especially now, and can't be explained properly this early in the tutorial.

> ## Exercise 4
>
> Take one of the programs you've written before, and find every lvalue in it, then every rvalue.
>
> Don't worry *too* much about memorizing this yet. A better understanding will come naturally as you use the language more, and as you learn the more advanced features that rely on those concepts.

## Const references

This appears contradictory at face value, but we do have const references in the language.

```cpp
void foo(const int &x)
{
    std::cout << x << '\n';
}
```
You can't assign to `x` in the function, so what's the point?

The point is that copying non-reference parameters can be expensive. Let's say you have a `std::vector<int>` with a few million elements, and want to pass it into a function. If you make a parameter of type `std::vector<int>`, calling the function will copy the entire vector, which is unnecessarily slow (unnecessarly, unless you actually need a copy for some reason), not to mention doubling the memory usage.

Using a reference `std::vector<int> &x` would get rid of the copying, but isn't a good solution either, because this function can no longer accept constant vectors (for no good reason, if it's not going to modify them), and also can't accept rvalue vectors (`foo({1, 2, 3});`, which is convenient and legal if the parameter is a non-reference vector; note that the lone `{1, 2, 3}` isn't a vector by itself, but can serve as an initializer for one), for the same reason as why you can't pass a literal into an `int &x` parameter (see above).

Const references solve all those problems. Passing arguments to them doesn't involve copying. Literals can be passed to them (and rvalues in general, see `foo({1, 2, 3})` above). Const variables can be passed too.

**Non-const references can only be initialized with non-const lvalues. Const references can be initialized with anything.**

## Const non-reference parameters

Okay, now what about `int foo(const int x)`?

When a parameter is `const` and isn't a reference, this is something that doesn't affect the caller at all.

This only matters inside the function. Do this if the function is particularly long and you're worried that you might accidentally modify a parameter that you're not supposed to. The caller won't see the change either way, but if *you* are reading the parameter multiple times in the function, then it might matter for you inside the function.

This concludes the part about passing parameters.

> ## Exercise 5
>
> Experiment with const reference and const non-reference parameters.

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

> ## Exercise 6
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

Returning references exposes you to a new kind of error that you probably didn't experience before.

What if you return a reference to a local variable?

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

Do you understand why this is UB? `int x;` only lives until the end of the function call, but the returned reference to `x` outlives `x`. We say that the reference is **dangling**, meaning it no longer refers to a valid object.

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
...since the returned reference ends up referencing `y` in `main` directly, not the parameter `x`. References never refer to other references, trying to do so makes it refer to the target object of that reference.

#### When else can references dangle?

Another case where this can happen is references to vector elements, when the vector is resized:
```cpp
std::vector<int> v = {1, 2, 3};
int &ref = v[1];
std::cout << ref << '\n'; // 2
v.push_back(4);
std::cout << ref << '\n'; // UB!
```
Notice that it isn't even a reference to an erased element. That would be UB for obvious reasons. The element we're referencing remains in the vector, but the act of resizing the vector can invalidate references to its elements.

This is given without an explanation for now. A proper explanation will be given in later chapters.

#### Dangling `const` references

`const` references are easier to dangle.

While `int &foo() {return 42;}` doesn't compile, because `42` is an rvalue, the same thing with a `const` reference does* compile:

```cpp
const int &foo()
{
    return 42;
}
```

This immediately dangles, so reading the result is always UB.

\* Newer C++ versions (C++26) make the above into a compilation error, since it's an obvious mistake.

> ## Exercise 8
>
> Experiment with UB caused by dangling references, and try to observe the incorrect values produced by it.
>
> Enable the [address sanitizer](/tooling/articles/recommended_compiler_flags.md) and see if it catches this error.
