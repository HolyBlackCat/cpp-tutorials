## Expressions and objects

The word "expression" always means a part of the source code.

When you print `(10 + 20) / 2`, the resulting number `15` (or the result of `+` which is `30`) are **not** expressions, because they aren't written directly in the source.

The numbers that are being manipulated (which exist in computer's memory while the program is running, not during compilation) are called **objects**.

Expressions refer to objects.

<sub>(There's some nuance to what can or can't be called an "object", and advanced readers might be yelling at their screens that in this specific context integers aren't really "objects". Yes. But this is just a nuance of the wording, and doesn't lead to any wrong conclusions, so there's no harm in pretending that they are, for simplicity's sake. Those nuances will be discussed and explained later.)</sub>




## Variables and objects

As we established in the previous chapter, the computer holds the results of computations in something called *objects* (e.g. when it computes `10 + 20` in the example above, the number `30` that it has to remember is an object, and so is `70`, and so are the original three numbers).<br/>
<sup>(Again, note to advanced reads: I'm [simplifying](./03_arithmetic.md#expressions-and-objects) things a bit here.)</sup>

And **variables are essentially named objects** that are manually created by the programmer (via declarations), as opposed to being created by the compiler automatically.

There's some nuance to this that will be explained later. Some variables can be unnamed, even though the obvious attempt at creating one (`int = 42;`) doesn't compile. And some variables are not objects, which will also be explained later.

Another nuance is that the word "variable" actually means a thing you have in your source code (during compilation), as opposed to an object that exists when your program is running ([proof](https://stackoverflow.com/q/79566041/2752075)), so variables *refer to* objects rather than actually *being* objects, but very few programmers think about or understand this distinction, and usually say "variables" to refer to the objects themselves.
