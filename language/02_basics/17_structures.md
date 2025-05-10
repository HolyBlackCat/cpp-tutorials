# Structures

Structures are a way to group together several variables. For example:

```cpp
#include <iostream>

int main()
{
    struct Monster
    {
        std::string name;
        int health;
    }; // Notice the `;`.

    Monster boss;
    boss.name = "Dragon";
    boss.health = 100;

    Monster henchman;
    henchman.name = "Gremlin";
    henchman.health = 20;

    std::cout << boss.name << '\n'; // Dragon
    std::cout << boss.health << '\n'; // 100
    std::cout << henchman.name << '\n'; // Gremlin
    std::cout << henchman.health << '\n'; // 20
}
```

Here,
```cpp
struct Monster
{
    std::string name;
    int health;
};
```
...is the **struct declaration** (and also a **struct definition**, this is another case where a thing is a declaration and a definition at the same time; the difference will be explained later).

Notice how it doesn't immediately create those two variables. You can't directly refer to `name` and `health`.

Instead, this makes `Monster` available as a type. You can then create variables of this type (`Monster boss;` and `Monster henchman;` above).

Each variable of the struct type then stores multiple values: one `name` string and one `health` integer.

`std::string name;` and `int health;` in this example are called the **data members** of the struct (or sometimes informally its "fields"). Structs can have other kinds of members, but this is for later.

Naturally, structures can serve as array/vector element types: `Monster monsters[4];`, `std::vector<Monster> monsters;`, etc. You can then use e.g. `monsters[0].name` to access a field of an element, and so on.

> ## Exercise 1
>
> Create a program that utilizes a vector of structs. For example, lets the user input as many elements as they like, and then prints them all.

## Struct names

Struct names are typically capitalized as `HelloWorld`. Some prefer `helloWorld` or `hello_world`, but this is rare (notably the standard library uses the latter). Avoid naming structs in `ALL_CAPS`, though some old libraries do this (even some parts of the standard library inherited from C). All caps names are typically used for something else, which will be explained in later chapters.

Struct names have same requirements as variable names. Revisit the chapter on variables if necessary.

## Differences with other languages

In some other programming languages (like Python), struct variables act in a funny way, where copying a struct and modifying the copy changes the original. C++ doesn't have such weird inconsistencies:
```cpp
Monster a;
a.name = "A";
a.health = 10;

Monster b = a;
b.health = 50;

std::cout << a.health << '\n'; // Still 10, not 50.
```
This would work only if `b` was a reference.

## The `;`

You might have noticed that a struct declaration ends with `;`, whereas before you never had to add `;` after `}`. Why is this?

This is because, as a shorthand, you can declare the variables of a struct type immediately after the struct:
```cpp
struct Monster
{
    std::string name;
    int health;
} boss, henchman;
```
...this is equivalent to:
```cpp
struct Monster
{
    std::string name;
    int health;
};
Monster boss, henchman;
```
So the `;` indicates the end of the variable list.

This shorthand usually isn't used, because people generally agree that it looks ugly.

The reason why this is allowed in the first place goes back to C, the C++'s predecessor. There you always had to spell `struct` before the struct name, even when declaring the variables: `struct Monster boss, henchman;` (which C++ still allows, but this again is almost never used). And `struct Monster {...} boss, henchman;` was seen as the longer form of the above (with `{...}` added), so declaring variables was allowed in this form too. Not the brightest move.

## Struct initialization

This is fairly intuitive.

```cpp
struct Monster
{
    std::string name;
    int health;
};

Monster boss = {"Dragon", 100};
```
You might also see this being spelled without `=`, which is legal, and works for arrays and vectors too. I do think omitting `=` looks better for structs, but it can lead to subtle bugs when done to vectors, so omitting it always isn't the brightest idea, despite what some people say. I hope to discuss this in later chapters. For now I'll use `=` everywhere for simplicity.

Like with arrays, if you skip some members, they will be zeroed: `Monster boss = {"Dragon"};` sets the `health` to zero, whereas in `Monster boss;` it would be initialized.

`Monster boss = {};` zeroes all the fields. Notice that for strings and vectors, "zeroing" doesn't do anything, as they are never uninitialized and are always empty by default.

Like vectors, but unlike arrays, the entire struct can be assigned to:
```cpp
Monster boss = {"Dragon", 100};
boss = {"Dragon (phase two)", 300};
```

### Designated initialization

In newer C++ you can perform what's called "designated initialization":

```cpp
Monster boss = {.name = "Dragon", .health = 100};
```
If this doesn't compile for you, you need to tell your compiler to use C++20 or a newer version. In Clang and GCC, this is done using `-std=c++20` (see [my tutorial on tooling](/README.md) for more details), and similarly with `/std:c++20` in MSVC.

You can omit members in aggregate initialization, and the omitted members are once again zeroed: `Monster boss = {.health = 100}` leaves the name empty.

Designated initializers must be in the same order as the members, e.g. `{.health = 100, .name = "Dragon"}` is a compilation error. But you can of course have "gaps" between members, e.g. given `struct A {int x, y, z;};`, you can do `A a = {.x = 10, .z = 20};`, but can't do `.z = 20, .x = 10`, because in the struct `x` is declared before `z`.

### In arrays and vectors

```cpp
Monster monsters[] = {
    {"Dragon", 100},
    {"Gremlin", 20},
};
```
```cpp
Monster monsters[] = {
    {.name = "Dragon", .health = 100},
    {.name = "Gremlin", .health = 20},
};
```
Mixing the two forms is allowed too:
```cpp
Monster monsters[] = {
    {.name = "Dragon", .health = 100},
    {"Gremlin", 20},
};
```
All the same things work with vectors.

## Default member initializers

You can set the default values for members:

```cpp
struct Monster
{
    std::string name = "Enemy";
    int health = 5;
};
```
They are used by the compiler if the custom initializers are not provided, instead of zeroing the respective member of leaving it uninitialized.

```cpp
Monster a;                   // .name = "Enemy",  .health = 5
Monster b = {};              // .name = "Enemy",  .health = 5
Monster c = {"Dragon", 100}; // .name = "Dragon", .health = 100
Monster d = {"Dragon"};      // .name = "Dragon", .health = 5
Monster e = {.health = 50};  // .name = "Enemy",  .health = 50
```

## Nested structs

Struct variables can be used as members of other structs:

```cpp
struct Health
{
    int body;
    int shield;
};

struct Monster
{
    std::string name;
    Health health;
};

Monster boss;
boss.health.body = 20;
boss.health.shield = 100;
```
The initialization works for those as you would expect (nested braces), try it.

Notice that the two structs are side by side, you are not required to put the entire struct declaration in another:
```cpp
struct Monster
{
    struct Health
    {
        int body;
        int shield;
    };

    std::string name;
    Health health;
};
```
This *is* legal, but all this achieves is changing the name of the inner struct (when used outside of the outer struct) from `Health` to `Monster::Health`, which isn't very useful (but can sometimes improve clarity).

```cpp
Health h; // Error, no such struct.
Monster::Health h; // ok
```

## Arrays and vectors as struct members

This naturally works, and there's not much to talk about here.

One fun fact is that while (whole) arrays can't be assigned, structs with arrays as members *can* be assigned (which assigns the array members elementwise).

> ## Exercise 2
>
> Make a little text game, where both you and your enemy have health and perhaps other stats. Use structs to represent them.
>
> Repeatly ask the player what they want to do (by inputting a string such as `attack`, `block`, etc), modify the stats accordingly, and have win/lose conditions.

## Constness

### Of whole struct variables

Variables of struct types can be constant. Then each member becomes constant too, must be initialized, and can't be changed after initialization.

```cpp
struct Monster
{
    std::string name;
    int health;
};

const Monster boss = {"Dragon", 100};
boss.health = 50; // Error, `boss` is constant.
```

### Of individual members

The individual members can be made `const` too:
```cpp
struct Monster
{
    const std::string name;
    int health;
};

const Monster boss = {"Dragon", 100};
boss.health = 50; // ok
boss.name = "Gremlin"; // Error, `name` is constant.
```

While this looks useful at face value, this is in fact highly annoying, because it makes the entire struct non-assignable:
```cpp
const Monster boss = {"Dragon", 100};
boss = {"Wyrm", 200}; // Error, `boss` isn't assignable because `name` isn't.
```

I recommend never using `const` members. We have more civilized ways of preventing individual members from being modifier, which will be discussed later.
