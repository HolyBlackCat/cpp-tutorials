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






## Nuances of terminology

Let's get the terminology right. What *exactly* do references refer to (bind to)?

You would probably say "variables", but that's not entirely correct:

```cpp
int arr[10];
int &ref = arr[3];
```
This is legal. The entire `arr` is a variable, but its individual elements are *not* variables, yet references can refer to them. (They are not variables because variables aren't *declared*. Only the entire array is declared, not the individual elements.)

The correct word here is an **object**. References refer to objects.

Variables (excluding reference variables) are objects, and some other things are objects too, such as array elements as shown above.

Objects are the things that exist while the program is running, they *have* values, and their values can change over time. E.g. in `int x = 10; x = 20;`, the value of variable/object `x` changes over time.

Even when you don't use variables, you're manipulating objects. A lone literal such as `10` is an object. The result of an expression like `10 + 20` is also an object. (I'm simplifying a bit, a more nuanced explanation will be given in later chapters.)

You'll notice that all those terms fall into two categories:

Things spelled in the source code | Things existing when the program is running
---|---
**Variable** — something that is declared and refers to an object.|**Object** — something that exists in memory while the program is running, and has a value.
**Literal** — a number or a string spelled in the source code, also refers to an object.|
**Expression** — a part of the source code that refers to an object: a literal, a variable name, an operator with its operands, etc.|

These are two separate universes: the things in the source code *refer to/designate* the things that actually exist when the program is running.

The words in the table are often used loosely. People will say "variable" and mean the object that it refers to, and so on. There is no point in being too pedantic here, as long as you're doing it intentionally.





Nope. Those are **temporary objects**, and references can't bind to them (exceptions are explained later).

This is purely an artificial limitation.

Temporary objects live until the end of the full-expression they were created in (revisit the beginning chapters if you forgot what full-expression is; roughly that means that they live until the end of the current line).

Typically you don't care about how long they exist, since you use their value and forget about them.
