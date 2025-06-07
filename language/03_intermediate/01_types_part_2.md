# Types (part 2)

You've already learned a few arithmetic types (those that hold numbers): `int`, `double`, `char`, and even `bool`.

This chapter provides a full list of those types, and explains how they work.

## Introduction to binary numbers

"Binary" is a numeral system (a system of writing numbers) that uses only 2 symbols (digits) instead of 10: `0` and `1`. Those are called **bits** (**bi**nary digi**t**s).

Binary is used often in programming because the computer memory is also made up of bits; of little cells that can be in one of the two states each (such as "has electrical charge" and "no charge"), which are typically used to represent `0` and `1`.

All numbers in your programs are stored in binary. For example, the integers (`int`, `char`, etc) are stored as follows:

```
 0 = 00000000
 1 = 00000001
 2 = 00000010
 3 = 00000011
 4 = 00000100
 5 = 00000101
 6 = 00000110
 7 = 00000111
 8 = 00001000
 9 = 00001001
10 = 00001010
11 = 00001011
12 = 00001100
13 = 00001101
14 = 00001110
15 = 00001111
...
```
The pattern should be obvious. Make sure you understand how to convert between decimal and binary numbers on paper. You can look up more newbie-friendly tutorials on this topic if needed.

Conversion from binary to decimal is done by multiplying the bits by the successive powers of two:

&nbsp; &nbsp; &nbsp; binary **1011** &nbsp; = &nbsp; **1**×2<sup>3</sup> + **0**×2<sup>2</sup> + **1**×2<sup>1</sup> + **1**×2<sup>0</sup> &nbsp; = &nbsp; **1**×8 + **0**×4 + **1**×2+ **1**×1 &nbsp; = &nbsp; 11

And the conversion to binary is done by trying to subtract powers of two from larger to smaller:

&nbsp; &nbsp; &nbsp; Is 11 ≥ 2<sup>3</sup> ? Yes, so write bit **1** and subtract &nbsp; 11 - 2<sup>3</sup> &nbsp; = &nbsp; 11 - 8 &nbsp; = &nbsp; 3<br/>
&nbsp; &nbsp; &nbsp; Is 3 ≥ 2<sup>2</sup> ? No, so write bit **0** and leave 3 as is<br/>
&nbsp; &nbsp; &nbsp; Is 3 ≥ 2<sup>1</sup> ? Yes, so write bit **1** and subtract &nbsp; 3 - 2<sup>1</sup> &nbsp; = &nbsp; 3 - 2 &nbsp; = &nbsp; 1<br/>
&nbsp; &nbsp; &nbsp; Is 1 ≥ 2<sup>0</sup> ? Yes, so write bit **1** and subtract &nbsp; 1 - 2<sup>0</sup> &nbsp; = &nbsp; 1 - 1 &nbsp; = &nbsp; 0 &nbsp; &nbsp; (this process always results in zero)

## Binary in C++

Prefix your integer literals with `0b` (or `0B`) to indicate that they are binary.

E.g. `0b1011` is exactly the same thing as `11`.

Now consider this example:

```cpp
int a = 11;
int b = 0b1011;
std::cout << a << '\n'; // 11
std::cout << b << '\n';
```
What does the last line print? `11` or `1011`?

The right answer is `11`. Variables don't remember if they were created using the binary notation or not. Internally they are all binary anyway. `a` and `b` are exactly the same.

You can print integers in binary like this:
```cpp
#include <bitset>
#include <iostream>

int main()
{
    std::cout << std::bitset<8>(11) << '\n';
}
```
Where `8` is how many bits you want to print. This prints `00001011`.

## The amount of bits in types

If you try to store very large numbers in an `int` variable, you'll see that it breaks down after a certain value. For example:

```cpp
int x;
x = 100'000'000;
std::cout << x << '\n'; // 100000000
x = 1'000'000'000;
std::cout << x << '\n'; // 1000000000
x = 10'000'000'000;
std::cout << x << '\n'; // 1410065408
```
(Notice `'` between the digits. It doesn't do anything and can be inserted for clarity, to make long numbers more readable. It is also not preserved in the variable, as you can see when printing it.)

(The exact threshold at which it breaks down can in theory vary, that's explained below, but most often it'll be as demonstrated here.)


This is because an `int` (and any other arithmetic type) is stored as a **fixed amount of bits**. If the binary number doesn't fit into that many bits, you see this behavior.

Ok, so how many bits exactly? `1'000'000'000` needs 30 bits, and `10'000'000'000` needs 34 bits, so it's somewhere between that.

You can get the exact amount using...

### The `sizeof` operator

Try printing `sizeof(int)`, and you'll most likely see `4`.

But why `4`, if we expected between 30 and 34?

The answer is that it's measured in **bytes**, not in **bits**. Bytes are packs of 8 bits, so an `int` is `sizeof(int) * 8` = `4 * 8` = `32` bits in total. Checks out.

The `sizeof` values can vary. How much they vary in practice is explained later in this chapter, but `sizeof(int) == 4` is pretty ubiquitous.

And note that `sizeof` values are fixed during compilation. If you send a compiled executable to your friend (and they can run it in the first place), they will always see the same value as you do.

Also note that in theory C++ permits a larger amount of bits in byte than 8, but this is even more rare than `sizeof(int) != 4`. You can see the accurate amount of bits in a byte by including `<climits>` and print `CHAR_BIT`. Typically `CHAR_BIT` is `8`. This assumption is so ingrained in most software that it makes zero sense checking `CHAR_BIT`, unless you actually have a piece of hardware on your hands that you know uses a different amount.

And lastly, you'll often see `sizeof` being used e.g. like this: `std::cout << sizeof 42 << '\n';`. It can apply not only to types, but also to expressions, as here. `sizeof 42` is exactly the same thing as `sizeof(int)`, because `42` has type `int`. And `sizeof(42)` is in turn the same as `sizeof 42`. The `(...)` can only be omitted for expressions, not for types, so `sizeof int` doesn't compile.

### The sign bit

Since we now know `int` is `32` bits wide (at least on our system), what the maximum number that it can hold?

Naively that would be all 32× `1` bits, which is equal to 2<sup>32</sup>-1 = `4'294'967'295`. But if you try this number, you'll notice that it doesn't fit too.

That's because one bit (called the **sign bit**) is reserved to indicate if this is a positive or negative number (remember that `int` can store negative numbers too). It is `1` for negative numbers and `0` for positive ones and for zero.

With that knowledge, the maximum positive number ends up being `0111...111` (`0` bit, then 31× `1` bits), which is equal to 2<sup>32-1</sup>-1 = `2'147'483'647`, which is correct. You can observe that it fits, and that number plus 1 no longer fits.

## Negative numbers in binary

So how are the the negative numbers represented in binary?

First of all, you can just do `int x = -0b1011;` and get `-11`, but that's obviously not how it's represented in memory, because the memory can only hold bits, not the magical `-` character.

As explained before, the positive numbers go from `000...000` = 0 up to `0111...111`. The leftmost zero bit (the sign bit) indicates that the number isn't negative. The negative numbers would be `1???...???`.

There are several methods to how negative numbers can be represented. Naively you could assume something like this: (This is called the "sign-magnitude" method.)
```
1000...000 = -0
1000...001 = -1
1000...010 = -2
```
But this is almost never used.

Instead what's used most often (and what is required by C++) is the **two's complement** method:

```
1111...111 = -1
1111...110 = -2
1111...101 = -3
1111...100 = -4
...
1000...001 = smallest negative number + 1
1000...000 = smallest negative number
```

As you can see, `-1` is represented as all `1` bits, and we count backwars from that all the way to `1000...000`, which is the smallest possible negative number. For 32-bit wide `int`, that would be -2<sup>32-1</sup> = `-2'147'483'648`.

Astute readers may have noticed that this doesn't match the maximum positive number, which is `2'147'483'647` (notice `7` at the end instead of `8`.) This is because we also have `0`, which is technically neither positive nor negative, but takes up the slot of one of the positive numbers.

Let's confirm this:
```cpp
#include <bitset>
#include <iostream>

int main()
{
    std::cout << std::bitset<sizeof(int) * 8>(-1) << '\n';          // 11111111111111111111111111111111
    std::cout << std::bitset<sizeof(int) * 8>(-2) << '\n';          // 11111111111111111111111111111110
    std::cout << std::bitset<sizeof(int) * 8>(-3) << '\n';          // 11111111111111111111111111111101
    std::cout << std::bitset<sizeof(int) * 8>(-4) << '\n';          // 11111111111111111111111111111100
    std::cout << std::bitset<sizeof(int) * 8>(-2147483648) << '\n'; // 10000000000000000000000000000000
}
```

And the reverse works too:
```cpp
int x = 0b1111'1111'1111'1111'1111'1111'1111'1101;
std::cout << x << '\n'; // -3
```

Interestingly, if you try to print this number directly (without saving it to a variable first), it'll print a large positive number (that doesn't fit into an `int`). And same for large decimal numbers. The reason for this is explained later in this chapter.

### Calculating negative binary numbers by hand

So how do you convert between a negative binary number and a decimal one by hand, on a piece of paper? There are several options:

* Add or subtract 2<sup>32</sup> (where `32` is the type width in bits).

  Let's say you want to convert `-3` to binary. &nbsp; -3 + 2<sup>32</sup> &nbsp; = &nbsp; -3 + 4294967296 &nbsp; = &nbsp; 4294967293, which in binary is `0b1111'1111'1111'1111'1111'1111'1111'1101`, which matches the examples above.

  And similarly in the opposite direction, you can convert that binary to decimal, and then subtract 2<sup>32</sup>.

* Flip all bits and add `1`. This has the same effect as the above, and can be easier to calculate, assuming you're able to perform addition directly in binary.

  Let's say you want to convert `-3` to binary. `3` in binary is `000...00011`. After flipping all bits (i.e. negating them; replacing `0` with `1` and vice versa) you get `111...11100`. That plus 1 is `111...11101`, which is `-3` in binary.

  The reverse is done in the same way. Starting from `111...11101`, we flip the bits to get `000...00010`, then add 1 to get `000...00011`, which is decimal `3`.

## Other numbering systems (octal and hexadecimal)

C++ also supports octal numbers (aka base-8, using digits `01234567`) and hexadecimal numbers (aka base-16, using digits `0123456789ABCDEF`, where `A` is 10, `B` is 11, etc, and `F` is 16).

Octal literals begin with `0`. E.g. `017` is octal and results in decimal `15`.

Hexadecimal literals begin with `0x` (or `0X`), e.g. `0x10EF` is hex and results in decimal `4335`. (Those can be in lowercase too, e.g. `0x10ef`.)

The octal ones are used rarely, but the hex ones are very popular, because two hex digits nicely correspond to one byte.

### Printing octal and hex numbers

The process of printing those is inconsistent with that for the binary.

```cpp
std::cout << std::hex << 4335 << '\n'; // 10ef
std::cout << std::oct << 15 << '\n'; // 17
```

**NOTE:** `std::hex` and `std::oct` affect all numbers printed after them, not just one. After you're done with them, use `std::dec` to switch back to decimal mode.

### Converting between octal/hex and decimal

The process of converting between those systems and the decimal is similar that for the binary:

&nbsp; &nbsp; &nbsp; 0x**10EF** &nbsp; = &nbsp; **1**×16<sup>3</sup> + **0**×16<sup>2</sup> + **E**×16<sup>1</sup> + **F**×16<sup>0</sup> &nbsp; = &nbsp; **1**×4096 + **0**×256 + **14**×16 + **15**×1 &nbsp; = &nbsp; 4335

&nbsp; &nbsp; &nbsp; (again, note that A=10, B=11, C=12, D=13, E=14, F=15)

And the reverse:

&nbsp; &nbsp; &nbsp; Divide 4335 by 16<sup>3</sup> (4335/4096), which results in **1** and remainder 239.<br/>
&nbsp; &nbsp; &nbsp; Divide 239 by 16<sup>2</sup> (239/256), which results in **0** and remainder 239. &nbsp; (as 239 is less than 256)<br/>
&nbsp; &nbsp; &nbsp; Divide 239 by 16<sup>1</sup> (239/16), which results in **14** (which is **E**) and remainder 15.<br/>
&nbsp; &nbsp; &nbsp; Divide 15 by 16<sup>0</sup> (15/1), which results in **15** (which is **F**) and remainder 0. &nbsp; (this process always results in zero)

Similarly for octal.

### Converting between octal/hex and binary

Notably, there's a shortcut for converting directly between hex and binary: each hex digit corresponds to 4 bits, and can be converted separately:

&nbsp; &nbsp; &nbsp; Starting with 0x**10EF**:

&nbsp; &nbsp; &nbsp; 0x**1** = 1 = 0b**0001**<br/>
&nbsp; &nbsp; &nbsp; 0x**0** = 0 = 0b**0000**<br/>
&nbsp; &nbsp; &nbsp; 0x**E** = 14 = 0b**1110**<br/>
&nbsp; &nbsp; &nbsp; 0x**F** = 15 = 0b**1111**<br/>

&nbsp; &nbsp; &nbsp; So `0x10EF` = `0b0001'0000'1110'1111`

And in reverse, by slicing the binary into groups of 4 bits, and converting each group to one hex digit. (Notice that slicing has to be done starting from the right: `0b1000011101111` -> `0b1'0000'1110'1111`, as opposed to `0b1000'0111'0111'1`, which would give wrong results.)

Similarly between octal and binary, but that uses groups of 3 bits rather than 4.

This concludes the explanation of numeral systems.

## Unsigned integers

As explained above, `int` can typically hold numbers between `-2'147'483'648` and `2'147'483'647` inclusive (between -2<sup>32-1</sup> and 2<sup>32-1</sup>-1 inclusive, where `32` is the bit width of `int`).

But what if we don't need negative numbers, and need to store larger positive numbers?

For that we have the type `unsigned int`. It has the same `sizeof` as `int`. It can't hold negative numbers, and uses the 32th bit (which is the sign bit in `int`) to be able to hold larger positive numbers. It can hold numbers between `0` and `4'294'967'295` inclusive (between 0 and 2<sup>32</sup>-1 inclusive). So, about twice as large positive number compared to `int`.

`unsigned int` can also be spelled as just `unsigned`, but people usually don't do this.

And `int` can be spelled as `signed int`, but it's also useless and people don't do this.

### When to use unsigned integers

Some people (primarily newbies) sometimes try to use `unsigned int` for *all* variables that can't be negative.

A good idea? Nope, quite a bad one. Try to only use `unsigned int` when you actually need to store large positive integers.

The reason to avoid it is that `unsigned int` often behaves unintuitively:

```cpp
unsigned int x = 3;
std::cout << -x << '\n';
```
What do you think gets printed?

`-x` still has type `unsigned int`, so it *can't* be a negative number. So it **wraps around** and produces -3 + 2<sup>32</sup> &nbsp; = &nbsp; 4294967293.

A similar thing would happen if you did `x -= 10;` (if `x < 10` and the result would have to be negative).

A similar wraparound happens to unsigned integers if you try to make them too large. In that case 2<sup>32</sup> is subtracted from the result to get them in range.

### Conversions from signed to unsigned

Another common pitfall is this:

```cpp
unsigned int x = 3;
int y = -2;
std::cout << (x > y) << '\n';
```

What do you think this prints?

Turns out that `>` (and `==` and other comparison operators) require the operands to be of the same type. And when one of them is unsigned and the other isn't, they are **both** made unsigned. As if by:

```cpp
unsigned int y_unsigned = y;
std::cout << (x > y_unsigned) << '\n';
```
This makes `y` wrap around, so `-2` becomes &nbsp; -2 + 2<sup>32</sup> &nbsp; = &nbsp; `4'294'967'294`, &nbsp; which is greater than `x = 3`, so the code above unexpectedly prints `0` (`false`).

This often happens in loops:
```cpp
int n = 3;
for (unsigned int i = 0; i < n; i++)
{
    // ...
}
```
This loop works fine most of the time, as long as `n` isn't negative. But if you make it negative, you'd expect the loop to not run at all, but instead it does run a bunch of times (`i < n` behaves as explained above).

### Conversions from unsigned to signed

The opposite conversion wraps around too.

If you try to convert a large `unsigned int` into an `int` and it doesn't fit, 2<sup>32</sup> is subtracted from it to make it fit.

For example:
```cpp
unsigned int x = 4'294'967'286;
int y = x;
std::cout << y << '\n'; // -10
```
`4'294'967'286` doesn't fit into an `int`, and &nbsp; 4294967286 - 2<sup>32</sup> &nbsp; = &nbsp; -10.

### Casts between signed and unsigned

Of course, in addition to the implicit conversions, you can also manually cast between `int` and `unsigned int`:

```cpp
unsigned int x = 4'294'967'286;
std::cout << int(x) << '\n'; // -10
```

But in the opposite direction:
```cpp
int x = -10;
std::cout << unsigned int(x) << '\n'; // Compilation error!
```
This doesn't compile. Not because the cast is somehow impossible, but simply because `unsigned int` is *two words*.

Instead this has to be spelled as `(unsigned int)x`. This is a different form of cast compared to what I taught before.

`(unsigned int)x` is called the **C-style cast** (because it's the only form the C language has). Of course `(int)x` is also legal, it works with any type.

`int(x)` is the **functional cast** (named like this because it looks like a function). This is the form that I taught before.

There is almost zero difference between the two, except as I explained the functional cast is a bit more picky about what types it accepts (can't work with types spelled with more than one word, like `unsigned int`).

Because there's almost zero difference, *both* are often collectively called "C-style cast" (when comparing them with *other* forms of casts that will be explained in future chapters).

Here's something else you'll see often: `(unsigned int)(x)` (used when `x` is an expression that needs to be grouped together to get the precedence right). Is this a functional cast or a C-style cast, what do you think?

It's a C-style one. Do you understand why? `(x)` is a valid expression, so `(unsigned int)(x)` follows the `(type)expression` pattern of the C-style cast. But `(unsigned int)` isn't a valid type (as e.g. `(unsigned int) x = 42;` isn't a valid variable declaration; you can't enclose types in parentheses like this), therefore `(unsigned int)(x)` doesn't fit the `type(expression)` pattern of the functional cast.
