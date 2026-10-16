# Integral types

You already learned about a few integral types (those that hold integers): `int`, `char`, and even `bool`.

C++ provides a lot of different integral types to choose from. They occupy different amount of memory, and can hold different ranges of values, so choosing a type is a tradeoff between range and memory usage.

## `unsigned int`

`int` can typically hold values between `-2147483648` and `2147483647`.

If you need to store slightly larger positive values *and* don't need to store negative values, you can trade the negative range for higher positive range: `unsigned int` can typically hold values between `0` and `4294967295` (the positive range is about twice as large).

## 

In both
