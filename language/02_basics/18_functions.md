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

First, notice that functions can only be declared outside of other functions. `void SayHello() {...}` is the **function declaration** (and simulaneously a **function definition**, like all other declarations you've seen so far, more on that later).

## Function names

Typically, functions are named as `SayHello` or `sayHello`. `say_hello` is rarely used, primarily only in the standard library. Hopefully no one names their functions in all caps.

Function names have the same restrictions on them as the variable names (revisit the chapter on variables if needed).

The only addition is that function names starting with `_` underscore are reserved and shouldn't be used. That is in addition to the names containing `__` (two underscores in a row) anywhere, or starting with `_` followed by an uppercase letter (which doesn't really matter now, since the lone starting `_` is already banned for functions).

Again, revisit the chapter on variables if you forgot what it means for a name to be reserved.

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

## Passing parameters

Currently `IncreaseX()` always increases `x` by `100`. We can change that by introducing a **parameter**:

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

You can have more than one parameter. Not much to say about this.

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

### Passing variables in parameters

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

Without `&`, the parameter variable is a copy of whatever variable/value you passed to it (initialized it with), just like any other variable. And making it a reference essentially makes it an "alternative name" for whatever variable (or something else) that you passed into it.

## More about references

Okay, so now what happens if we do `Increase(5, 100);`?

Same thing as what happens if you try this with a non-parameter reference variable:
```
int main()
{
    int &target = 5;
}
```
And that is a compilation error.

The reason for this should be intuitively obvious. If you have `void Increase(int &target, int value)`, this implies that you want to access value of the first argument after calling the function (or you wouldn't have made it a reference), so the compiler prevents you from using the function "incorrectly".

But...

## Value categories

...it is interesting what happens here on a technical level.

Why is `x` (given `int x;`) a valid initializer for a reference (or an argument for a reference parameter), but `5` isn't?

You'll also notice that
```
int x;
std::cin >> x;
```
...is valid, while `std::cin >> 5;` isn't.

Similarly, `x = 10;` is valid but `5 = 10;` isn't.

Clearly there's a more general rule at play here.

You might assume that those things are only legal for variables, but that's clearly not it, because array elements are not variables (only the entire arrays are), but can be `std::cin >>`ed into, assigned, etc. And the same is true for the struct members (`boss.health`), and those aren't really variables either (only the entire struct variable is a variable).

### lvalues and rvalues

The difference is that `x` (if declared as a variable), `array[0]` (an array element), `boss.health` (a struct member) are all **lvalues**, whereas `5` (a literal) is an **rvalue**.

"lvalue" and "rvalue" are **value categories**.

"lvalue" originally stood for "left value", meanaing it can be the lhs (left operand) of `=` assignment. But it can be the rhs too, of course.

"rvalue" stood for "right value", meaning it can only appear on the right side of an assignment.

Expressions that are variable names are lvalues, and their struct members and array elements are also lvalues.

Literals such as `5` are rvalues (notably except string literals, which are lvalues, for reasons that will be explained in a later chapter).

Not all lvalues are assignable, so "can be an lhs of assignment" isn't a proper definition of an lvalue. For example, `const` variables are still lvalues, but can't be assigned. Arrays can't be assigned either, despite being lvalues. Same for structs with const members, and those const members themselves.

A const variable can't be passed to an `int &target` parameter, or used with `std::cin >>`, so being an lvalue isn't the only requirement for those. Additionally they require the lack of constness.

### Summary

Lvalues (lvalue expressions) usually refer to things that will live for some time. Whereas rvalues (rvalue expressions) usually refer to things that are about to be destroyed. Variables live for some time. Literals such as `5` don't exist outside of the expression they're used in.

There's more to this classification than just "lvalues" and "rvalues". Rvalues are further split into two categories, but they aren't very important, especially now, and can't be explained properly this early in the tutorial.

## Const references

This appears contradictory at face value, but we do have const references in the language.

```cpp
void foo(const int &x)
{
    std::cout << x << '\n';
}
```
You can't assign to `x` in the function, so what's the point?

The point is that copying non-reference parameters can be expensive. Let's say you have a `std::vector<int>` with a few million elements, and want to pass it into a function. If you use a `std::vector<int> x` parameter, calling the function will copy the entire vector, which is unnecessarily slow (unless you actually need a copy for some reason), not to mention doubling the memory usage.

Using a reference `std::vector<int> &x` would get rid of the copying, but isn't a good solution either, because this function can no longer accept constant vectors (for no good reason, if it's not going to modify them). Additionally, having a reference prevents you from doing this like `foo({1, 2, 3});` (which is convenient and valid if the parameter is a non-reference vector), for the same reason as why you can't pass a literal into an `int &x` parameter (see above).

Const references solve all those problems. Passing arguments to them doesn't involve copying. Literals can be passed to them (and rvalues in general, see `foo({1, 2, 3})` above). Const variables can be passed too.

**Non-const references can only be initialized with non-const lvalues. Const references can be initialized with anything.**

This concludes the part about parameter passing.

## Returning values

Let's say we have a function that approximates the value of Pi:

```cpp
void ComputePi(double &pi)
{
    // TODO the actual algorithm!
}

int main()
{
    double value;
    ComputePi(value);
    std::cout << "Pi is approximately " << value << '\n';
}
```
This is fairly verbose, and kinda weird given that `ComputePi` doesn't care about the original value of `pi`, and only uses it to output the result.

Would be nice to be able to write just `double value = ComputePi();`, and indeed we *can* do that!

```cpp
double ComputePi()
{
    double pi;
    // TODO the actual algorithm.
    return pi;
}

int main()
{
    double value = ComputePi();
    std::cout << "Pi is approximately " << value << '\n';
}
```
Several things to note here. First, we now have `double ComputePi()` instead of `void ComputPi()`, as the function produces ("returns") a `double` instead of nothing (`void`). `double` here is the **return type** of the function.

Second, `ComputePi` now ends in `return ...;`. The value passed to `return` is the value that `main` receives in `double value = ComputePi();`. `return` can accept any expression, not necessarily a variable; you could do `return 3.1415;` if you wanted.

### What about `int main` then?

Astute readers might be wondering: if the thing to the left of the function name is its return type, why have we been doing `int main` rather than `void main`? Or if we use `int`, why are we not returning anything from `main`?

The C++ standard says that `main` must return an `int`. This `int` is used to indicate if the program has ran successfully (then you `return 0;`), or ended with an error (then you return some other number, which you can use to indicate the failure reason).

`main` defaults to returning `0` if you don't `return` something else. Only `main` does this. **Forgetting to return from any non-void function (except `main`) causes undefined behavior.**
