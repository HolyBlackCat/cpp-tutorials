# Function overloading

In C++ (unlike in C) functions can be **overloaded**. It means that several functions can have the same name:

```cpp
#include <iostream>

int sum(int a, int b) {return a + b;}

// The parameter names don't have to be the same. They are completely independent.
int sum(int a, int b, int c) {return a + b + c;}

int main()
{
    std::cout << sum(10, 20) << '\n';     // 30  - calls the first function
    std::cout << sum(10, 20, 40) << '\n'; // 70  - calls the second function
}
```

This is done purely for convenience (of not having to come up with unique names for similar functions).

Since there are several functions named `foo`, `foo` is said to be **overloaded**. Each individual `foo` function is called **an overload**, so in this example you have two overloads of `foo`.

There can be any number of overloads for a function, it's not limited to two.

The process of selecting a specific overload to call is called **overload resolution**. It is performed by the compiler at compile-time, based on the amount of arguments and their types (and some other things). In this context, the overloads being selected from are called "overload resolution candidates".

The overloads have to have a different number of parameters and/or different parameter types (or some other things), as overload resolution would be impossible otherwise. Having different return types is allowed, but you can't have overloads that *only* differ in return type ([see below](#not-by-return-type)).

## Overload resolution rules

The overload resolution process is [very complex](https://en.cppreference.com/w/cpp/language/overload_resolution.html), and memorizing the exact rules isn't useful. It's easier to get a feel for it by trying some examples.

Don't feel the need to memorize all of those. Understand the general idea, and revisit this chapter later if needed.

Here are the most important rules:

### By argument count

When the overloads have different the number of parameters, the selection process is fairly obvious. See the example above.

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

Note that deleting one of the overloads wouldn't cause a compilation error, as the remaining overload can still be called with the "wrong" argument type. The argument would then be converted to the parameter type, same as in e.g. `int x = 5.7;` (which results in `x` being `5`, revisit the [introduction to types](./08_types_1.md) if you forgot).

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

So does the compiler just count the conversions, and prefers the overload with less conversions? No.

It's not so simple. Consider the following example:

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

Roughly, for an overload to be selected, it needs to be **strictly better**, meaning it must be better in **some** way, and not worse in all other ways.

Here, `10` matches the second overload better, while `20` matches the first overload better. This already means that neither overloads is strictly better than the other, so the call is already ambiguous (causes a compilation error), regardless of what the other arguments are.

Compare it to the previous example (the one that compiles), where `10` and `30` match both overloads equally, and then `20` matches the first one better, therefore the first overload is a better match overall, and is selected.

This overall principle applies in a lot of different places in C++ (the compiler selecting the alternative that's *strictly better*, and complaining if there's none).

And of course, it's possible to pick arguments types that make this example not ambiguous. E.g. `a(1.0, 2, 3);` would call the first overload.

### Not by return type

You *can't* overload functions by the return type. I.e., you can't do this:

```cpp
int div_by_2(int x)
{
    return x / 2;
}

double div_by_2(int x) // Error here already. You can't have overloads that only differ in their return types.
{
    return double(x) / 2;
}

int main()
{
    int a = div_by_2(42);
    double b = div_by_2(42);
}
```

And the return type is ignored during overload resolution:

```cpp
int div_by_2(int x)
{
    return x / 2;
}

double div_by_2(double x)
{
    return x / 2;
}

int main()
{
    int a = div_by_2(42); // Calls first overload.
    double b = div_by_2(42); // Calls first overload because of the argument type. How the return value is used is ignored.
}
```

## Signature

The **signature** of a function is its properties that affect overload resolution. In this simple case the signature includes the function name and its parameter count and types, but not its return type.
