# Loops

Loops let you run a part of the source code multiple times.

The simplest kind of loop is the **`while` loop**:

```cpp
int x = 0;

while (x < 100)
{
    std::cout << "Enter a number: ";
    int y;
    std::cin >> y;

    x = x + y;
    std::cout << "The sum is: " << x << "\n";
}
```

This code asks for a number, then adds it to `x` and prints the sum. It keeps repeating this until the sum reaches `100` or higher.

Like `if`, `while` checks the condition and then executes the body if the condition is true. But then unlike `if` it keeps repeating this over and over again, until the condition becomes false.

Each repetition a loop is called an **iteration**.

Like in an `if`, in loops the `{...}` braces are optional if the body is a single statement. And like `if`, it has the same empty body pitfall (adding `;` after `while (...)` makes the body empty, which in the case of `while` typically makes the loop *infinite*, meaning the program gets stuck on it).

Any variables declared inside of the body are recreated on each iteration (meaning they lose their values). That's why we declared `x` outside of the loop.

> ## Exercise 1
>
> Write a program that utilizes the `while` loop.
>
> Do this for the two other kinds of loops described below too, after reading about them.

## The `do`-`while` loop

A rarely used variant of the `while` loop is the **`do`-`while` loop**:

```cpp
int x = 0;

do
{
    std::cout << "Enter a number: ";
    int y;
    std::cin >> y;

    x = x + y;
    std::cout << "The sum is: " << x << "\n";
}
while (x < 100);
```
The only difference here is that **the condition isn't checked before the first iteration**. The first iteration happens unconditionally.

After that, it proceeds exactly the same as the regular `while` loop.

In this case, the `do`-`while` is more appropriate because we know the condition is always going to be true before the first iteration.

Here braces can be omitted too, e.g.:
```cpp
do
    x = x + 10;
while (x < 100);
```
But the `;` after the `while (...)` part always has to be present.

## The `for` loop

This is the kind of loop you'll see most often.

```cpp
for (int i = 0; i < 5; i++)
{
    std::cout << i << "\n";
}
```
This prints `0` `1` `2` `3` `4`.

The `for` loop is just a glorified `while`. The above is equivalent to:

```cpp
{
    int i = 0;
    while (i < 5)
    {
        std::cout << i << "\n";

        i++;
    }
}
```
First of all, `i++;`. This operator is called an **increment**. Here it is equivalent to `i += 1;`, which is in turn equivalent to `i = i + 1;`. You could use any of those in the examples above, but `i++` is shorter to write. (Increments are explained in more detail in a later chapter.)

Second, notice the outer `{...}`. In the first example, `int i` only exists until the end of the loop, so if we didn't limit its scope with the `{...}` in the second example, it wouldn't be fully equivalent to the first one, because `int i` would outlive the loop.

To summarize,  `for (A; B; C) {D}` Does following

1. Runs `A`.
2. Checks condition `B`. If it's false, exits the loop and doesn't do anything else.
3. Runs the body `D`.
4. Runs `C`.
5. Goes back to step 2.

Any of `A`,`B`,`C` can be omitted, but the two `;`s must always be present. If the condition is omitted, it's assumed to be always `true`.

E.g. omitting all of them gives you `for (;;) {...}`, which is exactly equivalent to `while (true) {...}`, an infinite loop.

And `for (;B;) {...}` is equivalent to just `while (B) {...}`.

`A` doesn't have to be a variable declaration. It can be any expression, e.g. assignment:
```cpp
int x;
for (x = 0; x < 10; x++)
    std::cout << x << "\n";
```
This is used if you need to use the variable again after the loop (which otherwise would be destroyed when the loop finishes).

Notice that we're omitting the braces here.

## Nested loops

Naturally, loops can be nested:
```cpp
for (int i = 0; i < 3; i++)
{
    for (int j = 0; j < 3; j++)
        std::cout << " " << i * 3 + j + 1;

    std::cout << "\n";
}
```
This prints:
```
 1 2 3
 4 5 6
 7 8 9
```

The convention is to name loop variables `i`, `j`, `k`..., using the next letter for each nested loop.

Reusing the same letter will shadow the variable from the outer loop, just like in an `if` (as explained in the previous chapter).

If the loops are not nested, they can safely reuse the same variable name, because those variables only exist until the end of the loop:
```cpp
for (int i = 0; i < 3; i++)
    std::cout << i << "\n";

for (int i = 0; i < 3; i++)
    std::cout << i << "\n";
```

## The `break` statement

The **`break;` statement** allows you to stop a loop early, ignoring the condition. For example, let's extend the first example of this chapter a bit:

```cpp
int x = 0;

while (x < 100)
{
    std::cout << "Enter a number: ";
    int y;
    std::cin >> y;

    if (y < 0)
        break;

    x = x + y;
    std::cout << "The sum is: " << x << "\n";
}
```

Now this loop stops not only when the sum becomes `100` or larger, but also immediately when user inputs a negative number.

It works exactly the same way in `do`-`while` and in `for`.

You can of course use multiple `break;`s per loop.

In nested loops, `break;` only breaks out of a single loop (the most nested one), not all of them.

## The `continue` statement

The **`continue;` statement** skips the rest of the loop body and jumps straight to checking the condition again.

For example:
```cpp
int x = 0;

while (x < 100)
{
    std::cout << "Enter a number: ";
    int y;
    std::cin >> y;

    if (y < 0)
        continue;

    x = x + y;
    std::cout << "The sum is: " << x << "\n";
}
```

This version silently ignores any negative numbers. Try running it.

You can achieve the same effect with an `if`, so the only thing `continue` does is improving clarity.

In a `for` loop, `continue;` **does** execute the step `C` before checking the condition. To illustrate:
```cpp
for (int i = 0; i < 5; i++)
{
    if (i < 3)
        continue;
    std::cout << i << "\n";
}
```
This prints `3` `4`.

`continue;` still executes `i++`, despite skipping the rest of the loop body. If it didn't, the loop would get stuck forever on `i == 0`.

This means that the earlier example of rewriting a `for` in terms of a `while` doesn't hold if `continue;` is used.

Lastly, like `break;`, in nested loops `continue;` only affects a single loop (the most nested one), not all of them.

> ## Exercise 2
>
> Write a program that utilizes `break;`. Then another one for `continue;`.
