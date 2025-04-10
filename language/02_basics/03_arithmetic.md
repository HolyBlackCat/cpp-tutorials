# Doing arithmetic

"Arithmetic" means "simple math" (that with basic operations such as addition and multiplication).

You can use C++ to perform arithmetic (and do more advanced math too).

First of all, in addition to printing strings in quotes `std::cout << "Hello!";`
you can also print numbers without quotes: `std::cout << 42;`.

You can also print **expressions**:
```cpp
#include <iostream>

int main()
{
    std::cout << "The value is " << (10 + 20) / 2 << "\n";
}
```
This prints `The value is 15`. Notice that we're using multiple `<<` in one statement, this is fully equivalent to using multiple individual statements.

`(10 + 20) / 2` is an expression. Informally, **expressions are sequences of operands of operators**.

**Operators** are the symbols representing math operations, such as `+`, `-`, `*` (multiplication), `/` (division), and more.

**Operands** are what an operator applies to. In `(10 + 20) / 2`, the operands of `+` are `10` and `20`, and the operands of `/` are `(10 + 20)` and `2`.

The left operand of some operator will often be called an **lhs** (left hand side) and the right one an **rhs** (right hand side).

Experiment with printing different expressions. This already has some practical use.

## Expressions and objects

The word "expression" always means a part of the source code.

When you print `(10 + 20) / 2`, the resulting number `15` (or the result of `+` which is `30`) are **not** expressions, because they aren't written directly in the source.

The numbers that are being manipulated (which exist in computer's memory while the program is running, not during compilation) are called **objects**.

Expressions refer to objects.

## Expressions are composable

Expressions often have other expressions as their parts. In expression `(10 + 20) / 2`, the parts `(10 + 20)` and `2` are themselves expressions. And so are `10 + 20`, `10`, and `20`.

[![expression decomposition](../images/subexpressions.svg)](../images/subexpressions.svg)

Notice that individual numbers are also expressions.

The entire `std::cout << "The value is " << (10 + 20) / 2 << "\n"` (not counting the `;`) is also an expression. `<<`s are operators. Quoted strings are expressions.

Expressions that are parts of other expressions are called **subexpressions**.

Expressions that are not parts of other expressions are called **full expressions**.

`std::cout << "The value is " << (10 + 20) / 2 << "\n"` is a full expression, and every other expression in it is a subexpression.
