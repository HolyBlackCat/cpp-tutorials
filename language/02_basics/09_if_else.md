# `if` and `else`

All C++ programs you've written so far were doing computations.

Let's try something else. Let's say you want to write a program that, given person's age, prints whether they're legally allowed to buy alcohol or not. How would you do that?

```cpp
#include <iostream>

int main()
{
    int age;
    std::cout << "Enter age: ";
    std::cin >> age;

    if (age < 18) // Note! No `;` here, more on that later.
    {
        std::cout << "Too young!\n";
    }

    if (age >= 18)
    {
        std::cout << "Allowed!\n";
    }
}
```

The **`if` statement** takes a condition in `(...)`, and then a list of statements in `{...}`. The statements are executed only if the condition is true.

Some of the operators that can be used in the conditions are: `==` (equal), `!=` (not equal), `<`,`<=`,`>`,`>=` (less, less-or-equal, greater, greater-or-equal).

So above, the first condition is true if `age` is `17` or less, and the second condition is true if `age` is `18` or greater.

In situations like the above, when the second condition is the opposite of the first one, you can and should use `else` instead of the second `if`:
```cpp
if (age < 18)
{
    std::cout << "Too young!\n";
}
else
{
    std::cout << "Allowed!\n";
}
```
This has the same effect, except it can be a bit faster in some cases, since there's one less condition to check.

## Nesting

Of course you can nest `if` and `else`:

```cpp
if (age < 18)
{
    std::cout << "Too young!\n";

    if (age < 10)
        std::cout << "Wtf?\n";
}
```

## Combining several conditions

Use operators `&&` ("and") and `||` ("or") to combine together several conditions.

For example, `if (x < 10 || x > 20)` means "x is smaller than 10 or larger than 20".

And of course you can group those with `(...)`:
```cpp
if (x > 10 && (x < 20 || x == 100))
```
Can you tell for which values of `x` this is true? Experiment with it for a bit.

`&&` and `||` in C and C++ are said to be **"short-circuiting"**, meaning that the first operand is checked first, and then if that's enough to determine the final result (`false` for `&&`, `true` for `||`), then the second operand is skipped. This probably isn't very useful to you right now, because you have no way of observing that this actually happens, but this will be useful later.


## Conditions are expressions

Conditions are also expressions, like any other expression. For example, `if (10 + 20)` isn't a compilation error, and `int x = 10 < 20;` is also legal. Let me explain...

Comparison operators listed above (`==`,`!=`,`<`,`<=`,`>`,`>=`) result in 1 if the condition true and 0 if it's false.

To confirm this, try running `std::cout << (1 < 2) << "\n";` and observe that it prints `1` (meaning `1` is indeed less than `2`).

Here, notice the `(...)` which is necessary to avoid a compilation error, because otherwise this would be understood by the compiler as `(std::cout << 1) < (2 << "\n");` (which is meaningless), because `<<` has higher ["precedence"](https://en.cppreference.com/w/cpp/language/operator_precedence) (priority) than `<` (just like `*` has higher precedence than `+`, and `(1 + 2) * 3` needs the `(...)` to prevent `*` from being executed first).

### The `bool` type

But the type of that `1` or `0` (the result of a comparison) isn't `int`. It's a new type that you need to learn, **`bool`** (a "boolean", named after matematician [George Boole](https://en.wikipedia.org/wiki/George_Boole)).

A `bool` can only hold two values, 1 or 0. The correct way to spell them is `true` and `false` respectively. Just like `0` and `0.0` are different (`int` zero and `double` zero), `false` is also different (the `bool` zero). Same for `true`.

So while you can do this:
```cpp
int underage = age < 18;
if (underage)
{
    // ...
}
```
...which is equivalent to `if (age < 18) ...`, the more correct way to spell this is:

```cpp
bool underage = age < 18;
if (underage)
{
    // ...
}
```

In case it's not obvious, you can do `if (true)` (always executes) and `if (false)` (never executes). They aren't very useful, other than for teaching purposes.

### Conversions to and from `bool`

Intuitively, converting `bool` to a number (`int` or `double`) returns in `1` for `true` and `0` for `false`.

The reverse, converting a number to `bool`, results in `false` if the number is zero and `true` if it's not zero.

So given `int x;`, `if (x)` means the same thing as `if (x != 0)`, because the condition of an `if` is converted to a `bool`.

You can cast to `bool` too, `if (bool(x))` has the same effect as the above.

## Variables in braces

Variables can be declared in the `{...}` braces of an `if` or `else`:
```cpp
if (age < 18)
{
    int difference = 18 - age;
    std::cout << "You must be " << difference << " years older.\n";
}
```

Those variables are destroyed when leaving the `{...}` they were declared in:
```cpp
if (age < 18)
{
    int difference = 18 - age;
}

// This is a compilation error because `difference` no longer exists.
std::cout << "You must be " << difference << " years older.\n";
```
In this context the `{...}` are often referred to as the **scope** of the variable. As long as a variable exists, it's said to be **in scope**.

The period of time for which a variable exists is called its **lifetime**. When we leave the scope of a variable, its lifetime ends.

### Variable shadowing

As you already know, declaring two variables with the same name is a compilation error:
```cpp
int x;
int x; // Error, `x` is already declared.
```

But if one of them is inside of braces...
```cpp
int x = 10;
std::cout << x << "\n";

if (true)
{
    int x = 20;
    std::cout << x << "\n";
}

std::cout << x << "\n";
```
This compiles and prints `10` `20` `10`.

How this works should be fairly intuitive. The `int x = 20;` declaration is said to **shadow** (hide) the original `int x = 10;`, and now for as long as the second `x` exists (until we leave its scope), the name `x` refers to this second variable.

But the original variable continues to exist, and when the second one dies, the name `x` begins to refer to the original variable once again.

## Braces are optional

The braces `{...}` can be omitted if they only contain one statement.

For example:
```cpp
if (age < 18)
    std::cout << "Too young!\n";
```
...is the same as:
```cpp
if (age < 18)
{
    std::cout << "Too young!\n";
}
```

Note that the rules of variable scopes work the same way regardless. If you do `if (true) int x = 42;`, the variable won't be accessible outside of the `if`, despite us not using `{...}` here.

You can omit the `{...}` only for **one** statement. If you now try to do this:
```cpp
if (age < 18)
    std::cout << "Too young!\n";
    std::cout << "Go away!\n";
```
Only the first of the two statements will be guarded by the `if`. And the the second one will always run regardless of whether the condition is true.

As was already explained in the first chapter, in C++ the amount of whitespace (including at the beginning of lines) is ignored by the compiler (unlike in languages like Python).

So it would be more correct to write the last example as
```cpp
if (age < 18)
    std::cout << "Too young!\n";

std::cout << "Go away!\n";
```
...to clearly indicate this fact.
