# Functions and `const`

Let's explore how functions interact with all the other language features! Starting with simplest and least useful combination, with `const`.

## `const` parameters

Parameters can be `const`.

```cpp
#include <iostream>

void PrintValue(const int x)
{
    std::cout << x << '\n';
}

int main()
{
    PrintValue(42);
}
```

This isn't particularly useful.

This has zero impact on the caller (on what arguments can or can't be passed to this function).

Inside of the function this prevents you from modifying the parameter (changes to which wouldn't affect the caller anyway, since it's not a reference). The only point is preventing yourself from doing it accidentally. Most people don't add `const` to parameters.

Note that `const` references as parameters are a different beast and *are* useful, they are explained in future chapters.

## Returning `const`

```cpp
#include <iostream>

const int GetValue()
{
    return 42;
}

int main()
{
    PrintValue(42);
}
