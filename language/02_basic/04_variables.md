# Variables

Consider this expression: `(10 + 20 + 40) * (10 + 20 + 40)`.

The operands of the `*` operator are the same. Instead of writing the expression
twice, you can store its value in a **variable**:

```cpp
#include <iostream>

int main()
{
    int x = 10 + 20 + 40;
    std::cout << x << "\n";
}
```
This outputs `70`. And printing `x * x` would obviously output `4900`.

Here, `x` is a variable. Simply speaking, variables are values that have names.
(This is highly simplified. A more precise explanation is given in later chapters.)

## Variable declarations

`int x;` is a variable **declaration**. It declares that a variable exists. When
declaring a variable, you can also give it a value:
```cpp
int x = 42;
```
This is a variable **definition**.

All variables must be declared before use. The following is a compilation error:
```cpp
std::cout << x << "\n"; // Error, `x` isn't declared yet!
int x = 42;
```
It's recommended to declare variables as late in the code as possible, right
before they are needed, as that tends to result in more readable code

Trying to declare the same variable twice is a also compilation error:
```cpp
int x = 42;
int x = 43; // Error, `x` is already declared!
```

## Types

`int` is the **type** of the variable. It describes what kind of data the
variable stores. `int` means it stores an integer. Other types will be intoduced
in the later chapters. You can make string variables, fractional numbers and
more.

## Assignment

In C++ and most other programming languages, the values of variables can be
changed:

```cpp
int x = 10;
std::cout << x << "\n";
x = 20;
std::cout << x << "\n";
```
This outputs:\
`10`\
`20`.

Notice that modifying an existing variable is spelled differently (without
`int`) than declaring it. Unlike in math, `x = 20` is not an equation.
`x = ...` is an **assignment**, and what happens during declaration is **not**
an assignment, despite both being spelled with `=`. This is called
"initialization" and is explained in following chapters.

You can only assign to a variable that's already declared. The following is a
compilation error:
```cpp
x = 20; // Error, `x` isn't declared (yet)!
int x = 20;
```


## `=` is not symmetric

While in math `x = 20` and `20 = x` are equivalent, in C++ `20 = x` is a
compilation error. And so is `20 = int x;`.

In other words, the lhs of `=` must be a variable (some other things are allowed
too, but more on that later).

Something like `x * 2` isn't a valid lhs of `=` either, which means that the
compiler will not solve equations for you. You can't write `x * 2 = 100;` and
expect it to figure out that `x` is `50`. Since `x * 2` is not a variable, this
is a compilation error.

## Changing one variable doesn't automatically update another

This is another case where intuition often fails.

```cpp
int a = 10;
int b = a;
a = 20;
std::cout << b << "\n";
```
What does this output? Go and test it.

It outputs `10` because the value of `b` is computed when it's defined and
doesn't change after that, even if `a` changes.

## Referring to the variable itself in the assignment

This is legal:
```cpp
int x = 10;
x = x + 100;
```
The second line increases the value of `x` by `100`, resulting in `110`. Again,
assignment isn't an equation.

`x = x + 100`; can also be written as `x += 100;`. Similarly there are `-=`,
`*=`, `/=`, etc.

> ## Exercise 1
>
> Create a variable that holds your age, and print it. Name it appropriately.
>
> Make the program increase it a few times and print the updated values too.

## Valid variable names

What names (also called "identifiers") can you use for your variables?

Any combination of uppercase and lowercase letters, digits and `_` - the
underscore symbol that doesn't begin with a digit. If it begins with a digit,
it's a number.

Again, C++ is case-sensitive, so you can have `x` and `X` as two separate
variables.

Spaces are not allowed in identifiers, so to separate words people usually spell
them as `hello_world` or `helloWorld`. While `HelloWorld` is also an option,
it's usually not used for variables.

Additionally, you can't use words like `int` as variable names (`int int = 42;`
is illegal, as `int` is a type, which has special meaning). Identifiers with
special meaning are called **keywords**, and there are
[almost a hundred](https://en.cppreference.com/w/cpp/keyword) of them. You of
course don't need to remember all of them (at least not yet), but you should
know where to look. And your IDE should highlight them in a different color
than other identifiers, so they should be easy to notice.

Additionally, modern compilers accept identifiers in scripts other than latin:
```cpp
int μεταβλητός = 42;
```
However, these are rarely used.

### Reserved identifiers

Some identifiers are said to be **reserved**, meaning they mustn't be used for
your variables.

All identifiers that begin with two underscores in a row or an underscore
followed by an uppercase letter are reserved.

Don't use reserved identifiers.

> ## Exercise 2
>
> Experiment with different variable names. Confirm that you get a compilation
> error when trying to use a keyword as a variable name.
