# Computations

You can use C++ to perform computations.

In addition to printing strings in quotes `std::cout << "Hello!";`,
you can also print numbers without quotes: `std::cout << 42;`.

But more generally you can print the values of **expressions**:
```cpp
#include <iostream>

int main()
{
    std::cout << "The value is " << (10 + 20) / 2 << "\n";
}
```
This prints `The value is 15`. Notice that we're using multiple `<<` in one statement, this is the same as using multiple individual print statements.

`(10 + 20) / 2` is an expression. Informally, **expressions are sequences of operands and operators that produce a value**.

**Operators** are the symbols representing math operations, such as `+`, `-`, `*` (multiplication), `/` (division), as well as other kinds of operations we'll look at later.

Operators are applied to **operands**. In `(10 + 20) / 2`, the operands of `+` are `10` and `20`, and the operands of `/` are `(10 + 20)` and `2`.

The operands of a binary operator like `+` will often be called the **left-hand** and **right-hand side** ("lhs" and "rhs").

Experiment with printing the values of different expressions.

> ## Exercise 1
>
> Look up the number of days in each month, then add them together using C++. Confirm that they sum to 365.

## Expressions are composable

Expressions can be made up of other expressions. In the expression `(10 + 20) / 2`, `(10 + 20)` and `2` are themselves expressions, and so are `10 + 20`, `10` and `20`.

[![expression decomposition](../images/subexpressions.svg)](../images/subexpressions.svg)

Notice that individual numbers are also expressions.

We said earlier that `std::cout << "Hello, world!";` is a statement. It is also an expression. `<<` is the "stream insertion operator". Strings are expressions. Expressions can be statements.

Expressions that are parts of other expressions are called **subexpressions**.

Expressions that are not parts of other expressions are called **full expressions**.

`std::cout << "The value is " << (10 + 20) / 2 << "\n"` is a full expression, and every other expression in it is a subexpression.

> ## Exercise 2
>
> Write the first example from this chapter on a piece of paper. Circle every operand in it. Then cirlce (perhaps with a different color) eveyr expression in it.
