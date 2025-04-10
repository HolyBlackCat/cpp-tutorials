# Variables

Consider this expression: `(10 + 20 + 40) * (10 + 20 + 40)`.

Writing the repeating part twice manually isn't very convenient. We can make our job easier by using **variables**:

```cpp
#include <iostream>

int main()
{
    int x = 10 + 20 + 40;
    std::cout << x << '\n';
}
```
This prints `70`. And printing `x + x` would obviously give you `140`.

Here `x` is a variable.

The `int x = ...;` is a variable **declaration**, meaning it declares (creates) the variable. (Sometimes also called a variable **definition**. In this case there's no difference, but there are other things that are one but not the other, more on that later.)

`x + x` is not only more convenient to write than `(10 + 20 + 40) * (10 + 20 + 40)`, but will work faster, because `10 + 20 + 40` is computed only once instead of twice. (Though this doesn't actually matter in this simple case, because the compiler will most likely figure it out on its own and simplify the program to something like `std::cout << 140 << '\n';`. It will matter in more complex cases though.)

`int` is the **type** of the variable, it describes what kind of contents the variable stores. `int` means it stores an integer. Other types will be intoduced later in this tutorial.

## Variables in programming and in math

"Variable" in programming means something different than in math.

In math, if you do `x = 5` and then `x = 6`, this is a contradiction, because the variable can't have two different values.

But in most programming languages, variables can change over time. In C++, you can write following: (here and below I start omitting the includes and `int main(){...}` for brevity)

```cpp
int x = 10;
std::cout << x << '\n';
x = 20;
std::cout << x << '\n';
```
This prints `10` `20`.

Also in programming `=` isn't symmetric. Meaning `20 = x;` is illegal. The lhs must be a variable (or some other things, more on that later). (And as explained before, "lhs" means "left hand side", the left operand of an operator.)

This also means that the compiler will not solve equations for you. You can't write `x * 2 = 100;` and expect it to figure out that `x` is `50`. Since `x * 2` is not a variable, this a compilation error.

## How long does a variable exist?

The variable starts existing at the point of its declaration, and stops existing when leaving the `{...}` where it was declared (this is a slightly simplified explanation).

So the following is illegal:
```cpp
x = 42;                   // Compilation error, `x` wasn't declared yet.
std::cout << x << '\n';   // Another compilation error, `x` wasn't declared yet.

int x;   // This is too late now.
```

Notice that in C and C++, a variable must be declared before use. This is as opposed to e.g. Python, where simply assigning to a variable (`x = 42;`) implicitly creates it.


## Initialization

**"Initiailization" means "assigning the initial value".** ("Initial" meaning "first" value, which can be modified later.)

In C and C++ specifically, doing initialization means doing
```cpp
int x = 42;
```
as opposed to
```cpp
int x;
x = 42;
```

If you set a value when creating the variable, this is called initialization. (As in the first example.)

If you then modify the variable later (regardless of whether the variable was originally initialized or not), that modification is not initialization, it is an **assignment**. (As in the second example.)

But the word "initialization" is often used loosely. It's often used in the general programming sense of "assigning an initial value", so doing `int x;` and then `x = 42;` before using `x` will often called an "initialization" as well.

## Uninitialized variables

In some languages variables are zeroed by default, but not in C/C++ (at least not all types).

So what happens when you try to read an uninitialized variable?

```cpp
int x;
std::cout << x << '\n';
```

This is not a compilation error.

At runtime (when you run the program) this can produce an unpredictable value.

But this is not a suitable source of random values (e.g. for games), because, among other reasons, this can print the same value every time.

Some compilers let you enable additional checks that will turn those into runtime errors (if you're also following my tooling tutorial and use the Clang compiler, you can enable the [memory sanitizer](/tooling/articles/recommended_compiler_flags.md) to catch them).

The exact consequences of doing this will be discussed later. For now just know that you shouldn't do this.

### The nuances of the wording

First of all, `int x;` is typically called an uninitialized variable, but strictly speaking what happens to it is called "default-initialization" (despite it not setting any default value our case).

So no variables are ever truly *uninitialized*, they are at worst default-initialized.

This nuance is rarely useful in practice though, people will commonly say "uninitialized" to refer to those and it's completely fine.

Second of all, if you do this:
```cpp
int x;
x = 42;
std::cout << x << '\n';
```
This is of course completely fine. `x` is never initialized, only assigned, but using it after assignment is fine, and we don't call it "uninitialized" after the assignment (see the remark above on the word "initialization" being used loosely).
