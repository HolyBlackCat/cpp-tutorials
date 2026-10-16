# Declaration statements

We need to clean up some terminology.

As I've said before, the contents of `int main(){...}` (also called it's "body") are a sequence of *statements*.

In:
```cpp
#include <iostream>

int main()
{
    int x = 10 + 20;
    std::cout << x * 2 << "\n";
}
```

`int x = 10 + 20;` is a statement and `std::cout << x * 2 << "\n";` is also a statement.

Those are two different kinds of statements.

`std::cout << x * 2 << "\n";` is an **expression statement** ([that you're already familiar with](./03_expressions.md#expressions-are-composable)), meaning all it is is an expression (operators and operands) followed by a `;`.

`int x = 10 + 20;` is a different kind of statement, a **declaration statement**. In it, `10 + 20` is an expression and the rest is not.

## Ok, so what?

What does this give us? Why would you want to know this terminology?

Well, for one it tells you that since `int x = 10 + 20` is not an expression, it can't go where expressions go. Something like `std::cout << int x = 10 << "\n";` would give you a compilation error.

> ## Exercise
>
> Write a simple program on a piece of paper, e.g. the one from this chapter.
>
> Circle every statement in it.
>
> Then circle (perhaps with a different color) every expression in it. Refer to the previous chapters if you forgot what exactly expressions are.

## Syntax and semantics

The rules of the language describing how specific elements (of the source code) can be combined to form valid programs are called the **syntax** of that language. It is important to know the meaning of this word, as it gets thrown around a lot in programming.

For example, you could say that "C++'s syntax requires operands to be expressions" (there are exceptions from this rule, but that's for later).

Compilation errors caused by nonsensical constructs such as `std::cout << int x = 10 << "\n";` are sometimes called **syntax errors** (meaning they are caused by violating the C++ syntax rules).

Syntax is contrasted with **semantics**, which is the *meaning* associated with certain language elements. The syntax only describes which programs are *valid*, but doesn't describe what they do or their meaning (their semantics).

For example, you could say that "the semantics of `+` is that it adds two numbers together".
