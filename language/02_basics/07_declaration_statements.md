# Declaration statements

We need to clean up the terminology a bit.

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

`std::cout << x * 2 << "\n";` is an **expression statement**, meaning all it is is an expression (operators and operands), followed by a `;`.

`int x = 10 + 20;` is a **declaration statement**. In it, `10 + 20` is an expression and the rest is not.

## Ok, so what?

What does it give us? Why would you want to know this terminology?

Well, for one it tells you that since `int x = 10 + 20` is not an expression, it can't go where expressions go. Something like `std::cout << int x = 10 << "\n";` would give you a compilation error.

## Exercise

Write a simple program on a piece of paper, e.g. the one from this chapter.

Circle every statement in it.

Then circle (perhaps with a different color) every expression in it. Refer to the previous chapters if you forgot what expressions are.
