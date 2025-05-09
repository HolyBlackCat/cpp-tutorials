# `if` and `else`

All C++ programs you've written so far were doing computations.

Let's try something else. Let's say you want to write a program that, given person's age, checks if they're legally allowed to drink. How would you do that?

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

The **`if` statement** takes a condition in `(...)`, and then a list of statements in `{...}` (its **"body"**). The statements in the body are executed only if the condition is true.

Some of the operators that can be used in the conditions are: `==` (equal), `!=` (not equal), `<`,`<=`,`>`,`>=` (less, less-or-equal, greater, greater-or-equal).

So above, the first condition is true if `age` is `17` or less, and the second condition is true if `age` is `18` or greater.

## `else`

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

A lone `else` causes a compilation error, of course:
```cpp
int age;
std::cin >> age;

else // Error here.
{
    std::cout << "Allowed!\n";
}
```

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

Use operators **`&&`** ("and") and **`||`** ("or") to combine together several conditions.

For example, `if (x < 10 || x > 20)` means "x is smaller than 10 or larger than 20".

And of course you can group those with `(...)`:
```cpp
if (x > 10 && (x < 20 || x == 100))
```
Can you tell for which values of `x` this is true? Experiment with it for a bit.

`||` is an **"inclusive or"**, meaning it checks if **one or both** conditions are true, rather than exactly one condition (which would be "exclusive or", which C++ doesn't support directly).

`&&` and `||` in C and C++ are said to be **"short-circuiting"**, meaning that the first operand is checked first, and then if that's enough to determine the final result (if the first operand is false for `&&`, or true for `||`), then the second operand is not computed. This probably isn't very useful to you right now, because you have no way of observing what is or isn't computed, but this will be useful later.


## Negating conditions

Use **`!`** ("not") to negate conditions.

For example, `!(a > b)` ("a isn't larger than b") is (in most cases) equivalent to `a <= b`.

In this simple case you should just use `<=` directly, but it can be helpful in the more complex cases.

Another example:

```cpp
if (x > 0 && !(x == 10 || x == 20))
```
Do you understand what this does? Experiment with it a bit.


## Conditions are expressions

Conditions are also expressions, like any other expression. For example, `if (10 + 20)` isn't a compilation error, and `int x = 10 < 20;` is also legal. Let me explain...

Comparison operators listed above (`==`,`!=`,`<`,`<=`,`>`,`>=`) result in 1 if the condition true and 0 if it's false.

To confirm this, try running `std::cout << (10 < 20) << "\n";` and observe that it prints `1` (meaning `10` is indeed less than `20`).

Here, notice the `(...)` which is necessary to avoid a compilation error, because otherwise this expression would be understood by the compiler as `(std::cout << 10) < (20 << "\n");` (which is meaningless), because `<<` has higher ["precedence"](https://en.cppreference.com/w/cpp/language/operator_precedence) (priority) than `<` (just like `*` has higher precedence than `+`, and `(1 + 2) * 3` needs the `(...)` to prevent `*` from being executed first).

### The `bool` type

But the type of that `1` or `0` (the result of a comparison) isn't `int`. It's a new type that you need to learn, **`bool`** (a "boolean", named after matematician [George Boole](https://en.wikipedia.org/wiki/George_Boole) and his "boolean algebra", which deals only with two values, 0 and 1).

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

In case it's not obvious, you can do things like `if (true)` or `if (1)` (always executes), and `if (false)` or `if (0)` (never executes). They aren't very useful, other than for teaching purposes.

### Conversions to and from `bool`

Intuitively, converting `bool` to a number (`int` or `double`) results in `1` for `true` and `0` for `false`.

The reverse, converting a number to `bool`, results in `false` if the number is zero and `true` if it's not zero.

So, given `int x;`, `if (x)` means the same thing as `if (x != 0)`, because the condition of an `if` is converted to a `bool`.

You can cast to `bool` too, `if (bool(x))` has the same effect as the above.

## Variables in braces

Variables can be declared in the `{...}` braces, in the bodies of `if` and `else`:
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

// This is a compilation error because `difference` no longer exists,
//   as if it was never declared at all.
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

How this works should be fairly intuitive. The `int x = 20;` declaration is said to **shadow** (hide) the original `int x = 10;`, and for as long as the second `x` exists (until we leave its scope), the name `x` refers to this second variable.

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

As was already explained in the first chapter, in C++ the amount of whitespace (including at the beginning of lines) is ignored by the compiler (unlike in the Python language, for example).

So it would be more correct to write the last example as
```cpp
if (age < 18)
    std::cout << "Too young!\n";

std::cout << "Go away!\n";
```
...to clearly indicate this fact.

Everything above is also true for `else`. You can do this:

```cpp
if (age < 18)
    std::cout << "Too young!\n";
else
    std::cout << "Allowed!\n";
```

You can also have braces in one branch (`if` or `else`) and not another, though it does look ugly:
```cpp
if (age < 18)
{
    std::cout << "Too young!\n";
}
else
    std::cout << "Allowed!\n";
```
```cpp
if (age < 18)
    std::cout << "Too young!\n";
else
{
    std::cout << "Allowed!\n";
}
```

Now, can you guess what happens here:
```cpp
if (age < 18)
    std::cout << "Too young!\n";
    std::cout << "Go away!\n";
else
    std::cout << "Allowed!\n";
```
This is again a compilation error, because since the second `std::cout << ...`, isn't a part of `if`, this `else` is now dangling (not attached to any `if`), which as shown above is a compilation error.

And this should be obvious now, hopefully:

```cpp
if (age < 18)
    std::cout << "Too young!\n";
else
    std::cout << "Allowed!\n";
    std::cout << "Go on.\n";
```

This is equivalent to:
```cpp
if (age < 18)
    std::cout << "Too young!\n";
else
    std::cout << "Allowed!\n";

std::cout << "Go on.\n";
```

## Chaining `if` `else`

You'll often see things like this:

```cpp
if (age < 10)
    std::cout << "A\n";
else if (age < 20)
    std::cout << "B\n";
else if (age < 30)
    std::cout << "C\n";
else
    std::cout << "D\n";
```
Or equivalently:
```cpp
if (age < 10)
{
    std::cout << "A\n";
}
else if (age < 20)
{
    std::cout << "B\n";
}
else if (age < 30)
{
    std::cout << "C\n";
}
else
{
    std::cout << "D\n";
}
```

This (`else if`) isn't some special language feature, we're just omitting some braces here. After adding the braces back, you should see that it's entirely equivalent to:
```cpp
if (age < 10)
{
    std::cout << "A\n";
}
else
{
    if (age < 20)
    {
        std::cout << "B\n";
    }
    else
    {
        if (age < 30)
        {
            std::cout << "C\n";
        }
        else
        {
            std::cout << "D\n";
        }
    }
}
```

## Empty bodies

A common trap that newbies fall into is this:
```cpp
if (age < 18); // Note the `;`!
{
    std::cout << "Too young!\n";
}
```
Adding `;` here makes `if` seemingly ignore the condition and always execute the body. Why?

First of all, a lone `;` is a statement too, an **empty statement**.

So the above is equivalent to:
```cpp
if (age < 18)
    ;

{
    std::cout << "Too young!\n";
}
```
Which is in turn equivalent to:
```cpp
if (age < 18)
{
    ; // Does nothing.
}

{
    std::cout << "Too young!\n";
}
```
As you can see, adding `;` after `if (...)` gives it an empty body, same as `{}`.

Now the only question is what are those lone `{...}` braces after the `if`?

They do nothing, the contents are always executed. The only effect they have is acting as a scope for variables (variables declared inside still stop existing when exiting the braces).

Braces are rarely used like this intentionally (more often this is a typo). But in rare cases this can be useful to limit the scope of variables. Here's a small made-up example:
```cpp
int x;

{
    int y = 10 + 20 + 40;
    x = y * (y - 1);
}

// `y` doesn't exist anymore. We no longer need it.

std::cout << x << "\n";
```

And lastly, this common typo (`;` after `if (...)`) can be caught by your compiler automatically, if you enable compiler warnings. If you're also following my tooling tutorial, that is explained [here](/tooling/articles/recommended_compiler_flags.md#flags-to-catch-errors) (use `-Wall -Wextra` if you're using Clang or GCC).

## Terminology

Now I want to give the correct terminology for describing `if`/`else`.

As I said before, `int main() {...}` holds a list of statements. Therefore the entire `if (...) {...} else {...}` (or `if (...) {...}` if there is no `else`) is one big statement. This is a separate kind of statements, called **`if` statements** (unsurprisingly).

`{...}` braces with their contents are yet another kind of statements, called **compound statements**, or **block statements**. They are statements that contain other statements inside of them.

Therefore, `if` in general looks like this:
```cpp
if (expression)
    statement
```
or:
```cpp
if (expression)
    statement
else
    statement
```
And the ability to add `{...}` braces isn't some special feature of `if`, but a consequence of compound statements being a thing.

## Indentation

Let me digress for a moment.

You can see that in all examples above I've been adding whitespace to the beginning of lines in a certain manner that makes it look "good". If you're familiar with other programming languages, this is probably obvious.

Using the "correct" amount of whitespace at the beginning of lines is called **indentation**, and is something you should get used to.

C++ doesn't have any *official* rules of how things should be indented, and this tutorial shows one of the popular styles.

Some people put `{` at the end of the previous line, and some don't (like myself).

Different people use different number of spaces for each indentation level, 4 spaces is a popular option.

Indentation is usually performed by pressing the <kbd>Tab</kbd> key (instead of repeatedly hitting <kbd>Space</kbd>), but whether <kbd>Tab</kbd> inputs an actual Tab character or a bunch of spaces should be configurable in your IDE. It's matter of preference.

## Exercise

Create a program that lets you input the current temperature, and describes how hot or cold it feels. Include 3 or 4 different responses.
