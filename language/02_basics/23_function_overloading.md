# Function overloading

Functions in C++ (unlike in C) can be **overloaded**. Meaning several functions can have the same name:

```cpp
#include <iostream>

int sum(int a, int b) {return a + b;}

// The parameter names don't have to be the same. They are completely independent.
int sum(int a, int b, int c) {return a + b + c;}

int main()
{
    std::cout << sum(10, 20) << '\n'; // 30
    std::cout << sum(10, 20, 40) << '\n'; // 70
}
```

This is done purely for convenience (of not having to come up with unique names for similar functions).

Since there are several functions named `foo`, `foo` is said to be **overloaded**. Each individual `foo` function is called **an overload**, so in this example you have two overloads of `foo`.

There can be any number of overloads for a function, it's not limited to two.

The process of selecting a specific overload to call is called **overload resolution**. It is performed by the compiler at compile-time, based on the amount of arguments and their types (and some other things). In this context, the overloads being selected from are called "overload resolution candidates".

The overloads have to have a different parameter count and/or different parameter types (or some other things), as overload resolution would be impossible otherwise .

## Overload resolution rules

The overload resolution process is [very complex](https://en.cppreference.com/w/cpp/language/overload_resolution.html), and memorizing the exact rules isn't useful. It's easier to get a feel for it by looking at some examples.

Don't feel the need to memorize all of those. Understand the general idea, and revisit this chapter later if needed.

Here are the most important rules:

### By argument count

When the overloads have different number of parameters, the selection process is fairly obvious. See the example above.

### By argument types

The overloads can also have different parameter types:

```cpp
#include <iostream>

void describe(int x)
{
    std::cout << "this is an int\n";
}

void describe(double x)
{
    std::cout << "this is a double\n";
}

int main()
{
    describe(42); // Prints `this is an int`.
    describe(42.0); // Prints `this is a double`.
}
```

Note that deleting one of the overloads wouldn't cause a compilation error, as the remaining overload can still be called with the "wrong" argument type. The argument would be converted to the parameter type, same as in e.g. `int x = 5.7;` (which results in `x` being `5`, revisit the introduction to [types](./08_types.md) if you forgot).

As you can see, overload resolution prefers to avoid type conversions.

### "Strictly better" overloads

Now the next example:
```cpp
void a(double x, int y, int z)
{
    std::cout << "first\n";
}

void a(double x, double y, int z)
{
    std::cout << "second\n";
}

int main()
{
    a(10, 20, 30);
}
```
This will call the first overload. Do you understand why?

Calling the first one requires converting `10` to `double`. Whereas calling the second one requires **two** conversions, `10` to `double` and `20` to `double`. Therefore the first one is a better match.

So does the compiler just count the conversions, and prefer the overload with less conversions?

No, it's not so simple. Consider the following example:

```cpp
void a(double x, int y, int z)
{
    std::cout << "first\n";
}

void a(int x, double y, double z)
{
    std::cout << "second\n";
}

int main()
{
    a(10, 20, 30);
}
```
This is now a compilation error.

Roughly, for an overload to be selected, it needs to be **strictly better** (better in some aspect, and not worse in all other aspects).

Here, `10` matches the second overload better, while `20` matches the first overload better. This already means that neither overloads is strictly better than the other, so the call is already ambiguous (causes a compilation error), regardless of what other arguments are there.

Compare it to the previous example (the one that compiles), where `10` and `30` match both overloads equally, and then `20` matches the first one better, therefore the first overload is a better match overall, and is selected.

This overall principle applies in a lot of different places in C++ (the compiler selecting the alternative that's *strictly better*, and complaining if there's none).
