# Variables

Consider this expression: `(10 + 20 + 40) * (10 + 20 + 40)`.

Spelling the repeating part twice manually isn't very convenient. You can make your job easier by using **variables**:

```cpp
#include <iostream>

int main()
{
    int x = 10 + 20 + 40;
    std::cout << x << "\n";
}
```
This prints `70`. And printing `x * x` would obviously give you `4900`.

Here `x` is a variable. Simply speaking, variables are values that have names. (This is highly simplified. A more precise explanation is given in later chapters.)

## Variable declarations

The `int x = ...;` is the variable **declaration**, it declares (creates) the variable. (Sometimes it's also called the variable **definition**. In this case there's no difference, but there are other things that are one but not the other, more on that later.)

All variables must be declared before use. The following is a compilation error:
```cpp
std::cout << x << "\n"; // Error, `x` isn't declared (yet)!
int x = 42;
```
It's recommended to declare variables as late in the code as possible, right before they are needed, as that tends to result in cleaner code.

Trying to do declare a variable twice (with the same name) is a also compilation error:
```cpp
int x = 42;
int x = 43; // Error, `x` is already declared!
```

## Types

`int` is the **type** of the variable, it describes what kind of contents the variable stores. `int` means it stores an integer. Other types will be intoduced in the later chapters. You can make variables store pieces of text ("strings"), fractional numbers, and more.

## Assignment

In C++ (and most other programming languages) the values of variables can be changed over time:

```cpp
int x = 10;
std::cout << x << "\n";
x = 20;
std::cout << x << "\n";
```
This prints `10` `20`.

Compare this to math, where `x = 5` `x = 6` would be a contradiction, as there each variable can only have one value.

Notice that modifying an existing variable is spelled differently (without `int`, which is the type) compared to declaring it. `x = ...` is called an **assignment**, and what happens during declaration is **not** an assignment, despite both being spelled with the `=` (it's called "initialization" and is explained in more detail the next chapters).

You can only assign to a variable that's already declared. The following is a compilation error:
```cpp
x = 20; // Error, `x` isn't declared (yet)!
int x = 20;
```
In some other languages (such as Python) assigning to a variable automatically creates it, and there are no declarations. This isn't the case in C++.

## `=` is not symmetric

This is another thing that's different about variables in math and in most programming languages.

While in math `x = 20` and `20 = x` are equivalent, in C++ `20 = x` is a compilation error. And so is `20 = int x;`.

In other words, the lhs of `=` (lhs meaning "left hand side", the left operand) must be a variable (some other things are allowed too, but more on that later).

Something like `x * 2` isn't a valid lhs of `=` too, which means that the compiler will not solve equations for you. You can't write `x * 2 = 100;` and expect it to figure out that `x` is `50`. Since `x * 2` is not a variable, this a compilation error.

## Changing one variable doesn't automatically update another

This is another case where intuition often fails newbies.

```cpp
int a = 10;
int b = a;
a = 20;
std::cout << b << "\n";
```
What does this print? Go test it.

It prints `10` because the value of `b` gets computed when it's initialized (or assigned), and then doesn't change after that even if `a` changes.

## Referring to the variable itself in the assignment

This is legal:
```cpp
int x = 10;
x = x + 100;
```
The second line increases the value of `x` by `100`, resulting in `110`.

This is another case where variables in C++ are unlike those in math, as in math `x = x + 100` is an unsolvable equation.

`x = x + 100`; can also be written as `x += 100;`. Similarly there are `-=`, `*=`, `/=`, etc.

> ## Exercise 1
>
> Create a variable that holds your age, and print it. Name it appropriately.
>
> Make the program increase it a few times and print the updated values too.

## Valid variable names

What names (also called "identifiers") can you use for your variables?

Anything that consists of `A-Z`, `a-z` letters (uppercase and lowercase), `0-9` digits, and `_` the underscore symbol.

Additionally, the names must not begin with a digit.

C++ is case-sensitive, meaning you can have `x` and `X` as two unrelated variables.

Spaces are not allowed in identifiers, so to separate words people usually spell them as `hello_world` or `helloWorld`. While `HelloWorld` is also an option, it's usually not used for variables. And hopefully nobody names their variables in `ALL_CAPS`.

Additionally, you can't use words like `int` as variable names (`int int = 42;` is illegal), hopefully for obvious reasons. (As `int` has a special meaning, allowing it as a variable name would be too confusing.) Those words with special meaning are called **keywords**, and there's [almost a hundred](https://en.cppreference.com/w/cpp/keyword) of them. You of course don't need to remember all of them (at least not yet), but you should know where to look. And your IDE should highlight them in a different color than other text, so they should be easy to notice.

Additionally, modern compilers accept identifiers in languages other than English too (`int μεταβλητός = 42;`) but you should avoid them to not confuse non-english-speaking readers of your code.

### Reserved identifiers

Some names are said to be **reserved**, meaning they shouldn't be used for your variables.

Such are any names that contain `__` (two underscores in a row), or begin with `_` followed by an uppercase letter.

Most often you can get away with using them, but *some* reserved identifiers will cause errors. For example, `int __restrict = 0;` doesn't compile, because all modern compilers give a special meaning to `__restrict` (which I won't explain right now), essentially treating it as a keyword.

Avoid using the reserved identifiers.

> ## Exercise 2
>
> Experiment with different variable names. Confirm that you get a compilation error when trying to use a keyword as a variable name.
