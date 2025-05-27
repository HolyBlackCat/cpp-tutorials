# More types

Declaring variables as `int x = ...;` creates integer variables (of type `int`).

Those can only store whole numbers. If you try to do `int x = 5.7;`, you'll get just a `5`. The fractional part is discarded.

To store fractional numbers, we need a different type - `double`.

```cpp
int a = 5.7;
double b = 5.7;
std::cout << a << "\n"; // 5
std::cout << b << "\n"; // 5.7
```

`double` stands for "double precision" (compared to a less precise kind of fractional numbers, this will be explained in more detail later).

There are many more other types, which will also be explained later.

## Types of expressions

Not only variables have types, expressions do to. A lone `5` is an `int`. A lone `5.7` is a `double`. Notably `5.0` is also a `double`, because of the `.`.

`5 + 10` is an `int`, as expected, and `5.7 + 100.1` is a `double`. And `5.0 + 10.0` is also a `double`.

The fun begins when you start mixing the types. `100 + 5.7` gives `105.7` and has type `double`. It makes sense that the larger type is selected, so we're able to store the fractional part.

Notably `57 / 10` is an `int` (any operation on two `int`s gives an `int`), which means the fractional part is **truncated** (removed) and it gives just `5`.

What do you need to do to get `5.7` here? Think for a moment.

You make one or both operands a double by adding the `.`, e.g. `57.0 / 10` or `57 / 10.0` or `57.0 / 10.0`.

This however is not an option if both operands are variables:
```cpp
int x = 57, y = 10;
std::cout << x / y << "\n"; // 5, now what?
```
Enter...

## Type casts

```cpp
int x = 57, y = 10;
std::cout << double(x) / double(y) << "\n"; // 5.7
```

In this example we use **type casts** (or simply **"casts"**) to change the types of the expressions from `int` to `double`, so the division results in a fractional number.

Those casts have no lasting effects on the variables themselves, so if after the code above you print `x / y`, you'll still get `5`.

If you cast a `double` back to an `int`, it truncates (removes) the fractional part, as before.

Casts are also called **explicit type conversions**. This is as opposed to the **implicit type conversions**, that happen e.g. in `int x = 5.7;` or even in `double x = 5;` (though the latter mostly goes unnoticed, as it doesn't change the value).

There are many different kinds of casts that will be explained later. This is the simplest one.

## Common errors

Newbies often try to do this:
```cpp
double x = 57 / 10;
std::cout << x << "\n"; // 5
```
...and are confused why this doesn't print `5.7`. Do you understand why?

An `int` divided by an `int` always results in an `int`, which in this case is then converted to a `double`. It doesn't matter what the result of the division is used for, it doesn't affect how the division itself is performed.

> ## Exercise
>
> Create a program that asks the user for a temperature in Celsius and converts it to Fahrenheit, and the reverse. Google the conversion formulas if needed.
