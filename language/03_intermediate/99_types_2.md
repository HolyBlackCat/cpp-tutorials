
There are several different ways to spell a cast. With the types you've already learned there's zero difference between them.

A cast can also be spelled as `(double)x`, there's almost no difference between the two. There are other forms of casts that will be explained in the future chapters.








## What exactly are types?

Every value in a computer is stored as a sequence of **bits**.

A bit is the smallest part of computer memory. It can be in two states (e.g. "has electrical charge" or "has no charge" for some types of memory). A bit also mean the *amount of information* stored in one bit of memory.

**A type describes how many bits a value occupies, and what those bits mean**.

## Sizes of types

The above means that every variable (while it exists) uses up a certain amount of your memory (RAM).

How much memory exactly? You can figure out using the `sizeof` operator. Try printing `sizeof(int)` or `sizeof 42`. Most likely you will get `4` (more about the possible values is explained below). Those two forms are equivalent; when given an expression as opposed to a type, `sizeof` acts on its type (and `(...)` are optional for expressions).

What is `4`? The result of `sizeof` is measured in **bytes**. A byte is `8` bits, so this gives us `32` bits total for an `int`.

On some exotic computers bytes can have a different size than `8` bits. You can check it on yours by doing `#include <climits>` and then printing `CHAR_BIT`, which will most likely give you `8`.

Note that **type sizes are fixed during compilation**. So if you send the executable to your friend and they can run it at all, it will have the same type sizes as it had for you when you compiled it. Typically the sizes are determined by the combination of the OS and the CPU type (aka "CPU architecture": most desktops run x86, smartphones and modern Macs run Arm, etc).

## Limits of types

Each type occupies a certain number of bits, and there are only so many different combinations of bit values (of 0s and 1s). E.g. 32 bits gives you 2<sup>32</sup> = 4294967296 different combinations.

This means each type can only hold a finite range of numbers.

With 4294967296 different combinations, you'd think the range for `int` would be from 0 to 4294967295, but since it can also store negative values, that range is halved to -2147483648 ... 2147483647.

Astute readers might notice that the positive bound is one less than the negative one. That is because we also have `0`, which is neither positive nor negative, and by convention it uses up one positive number slot.

You can confirm those bounds by adding `#include <limits>` and printing `std::numeric_limits<int>::min()` and `std::numeric_limits<int>::max()`.

You might be wondering what happens when the value doesn't fit into a type, and that's explained later in this chapter.

## What other types are there?

You don't have to remember all of them.

### Unsigned types

First of all, **`unsigned int`**.

It can't store negative numbers, but the positive limit is approximately doubled, to 4294967295.

The normal `int` is a shorthand for `signed int`, but `signed` is usually omitted.

### Various integer types

Now the full list of integer types. Buckle up.

Type | Typical size<br/>(in bytes) | Possible size<br/>(in bytes)
---|---|---
`signed char`|`1`|always `1`
`short int`|`2`|`2` or larger
`int`|`4`|`2` and larger, and not smaller than `short int`
`long int`|`4` on Windows,<br/>`8` elsewhere|`4` or larger, and not smaller than `int`
`long long int`|`8`|`8` or larger, and not smaller than `long int`

This is a hot mess.

All of those are signed. They can hold values between -2<sup>N-1</sup> and 2<sup>N-1</sup>-1, where N is the type size in bits. You can use `std::numeric_limits` (as explained above) to check those limits.

Astute readers have noticed that I've spelled out `signed` only for `char`, even though I previously said that `int` and `signed int` are the same. The issue with `char` is that it (and only it) can be either signed or unsigned by default (which again depends on compiler settings and is fixed during compilation). So if you're planning to store numbers in `char`, you should explicitly mark it as `signed` or `unsigned`. There are other uses of it where the signedness doesn't matter, more on that in the next chapters.

Usually `char` is signed, but you shouldn't rely on it.

Next, in `short int`, `long int`, and `long long int`, the `int` part is optional and usually isn't spelled (so they become `short`, `long` and `long long`). The `int` is also optional in `unsigned int` and `signed int`, but it's usually not omitted from those, as this makes things hard to read.

Next, each of those types has an `unsigned` counterpart: `unsigned char`, `unsigned short` (`int`), `unsigned int`, `unsigned long` (`int`), `unsigned long long` (`int`). Again, the `int` part is optional.

Next, as you can see the specific type sizes are not guaranteed, even though in practice they are usually the same everywhere (other than `long`, which makes it practically useless). And as said previously, those sizes are fixed during compilation based on the OS and the CPU type, so if you send a compiled program to a friend and they can run it at all, there is no chance of the types having different sizes.

Enter...

### Fixed-size integer types

If you `#include <cstdint>`, you get access to fixed-width integer types:

Signed|Unsigned
---|---
`std::int8_t`|`std::uint8_t`
`std::int16_t`|`std::uint16_t`
`std::int32_t`|`std::uint32_t`
`std::int64_t`|`std::uint64_t`

Those are not some new types. They act as alternative names to appropriately sized built-in types (listed in the previous section). E.g. `std::int16_t` is typically `short`.

### How to choose the types?

This is a lot to choose from.

Typically, the rule of thumb is to use `int` by default
