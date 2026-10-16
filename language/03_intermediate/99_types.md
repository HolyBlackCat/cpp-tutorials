# Types

Try printing `10` and `10.0`. You should see the same number, which makes sense, since they are in fact the same number (`.0` is useless here and doesn't get printed).

Now try printing `57 / 10` and `57.0 / 10.0`, you should see `5` and `5.7` respectively.

Why the difference?

The values of both operands are seemingly the same, so there must be something else different about them.

They have different **types**: `10` is an integer (has type `int`) and `10.0` is a real number ("real" is a math term for "has a fractional part") (has type `double`, meaning "double precision" real number, more on that below).

In C and C++, types are always determined at compile-type (therefore C and C++ are said to be "statically typed"). I will explain the nuances of this in the next chapters.

## Objects and types

As [was briefly mentioned before](03_arithmetic.md#expressions-and-objects), when evaluating expressions (calculating their value), the computer needs to store the temporary results in its memory, and those stored things are called "objects".

For example, to compute `10 + 20 + 40`, the computer needs to remember `30` produced by the first `+`, to then add `40` to it, and this `30` is an object (and so is the final `70`, and also the `10`,`20`,`40` themselves).



For example,
