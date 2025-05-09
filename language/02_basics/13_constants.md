# Constants

Let's look at the example from the Arrays chapter again:
```cpp
int arr[5];

for (int i = 0; i < 5; i++)
    std::cin >> arr[i];

for (int i = 0; i < 5; i++)
    std::cout << arr[i] << "\n";
```

Here we're repeating `5` three times. Which isn't great, because if we decide to change the array size, we now have to go over the entire program and update all the `5`s.

You'd think creating a variable size to hold the array size would help...
```cpp
int size = 5;
int arr[size];

for (int i = 0; i < size; i++)
    std::cin >> arr[i];

for (int i = 0; i < size; i++)
    std::cout << arr[i] << "\n";
```
But as explained before, the array size must be fixed at compile-time, so `int arr[size];` is a compilation error.

Enter **constants**:

```cpp
const int size = 5;
int arr[size];

for (int i = 0; i < size; i++)
    std::cin >> arr[i];

for (int i = 0; i < size; i++)
    std::cout << arr[i] << "\n";
```

Adding `const` creates a **constant**, also known as a **constant variable** (yes, the name sounds self-contradictory).

This now compiles.

Constant variables can't be changed: `size = 42;` or `std::cin >> size;` would give you a compilation error.

Since the compiler can now clearly see that the array size is fixed, the example above compiles. You might be wondering why it can't see that with a non-constant variable. While this specific case looks simple, in more complex cases analyzing this would be impossible, so the compiler doesn't try at all.

## Exercise 1

Take one of the examples from previous chapters that uses an array, and modify it to use a constant for the array size. Modify any loops over that array to use that constant for the size too.

## Other uses of constants

You don't always use constants purely to make the compiler happy.

You can also use them to help yourself avoid mistakes (by accidentally modifying wrong variables).

## Exercise 2

One of previous exercises was to make a program to convert the number of days to the number of minutes.

Modify it to make the number of minutes in one day a constant, and use the constant in the calculation. Name the constant appropriately.

## Compile-time constants

```cpp
int x;
std::cin >> x;
const int size = x;
int arr[size];
```
What do you think happens here?

`const int size = x;` is in fact legal. But `int arr[size];` after that is illegal (a compilation error, again assuming the compiler is configured correctly).

So this means there are two diffent kinds of constants, the compile-time ones (which work as array sizes), and the non-compile-time ones.

I hope to explain this in more detail in later chapters, if I have time.


## Literals

Unnamed constants in the source code are called **literals**.

For example, when you do `int x = 42;`, the `42` is an **integer literal**.

`10 + 20` is not a literal. Only its parts, `10` and `20` are literals.

Variables (`const` or not) are also **not** literals.

Like "expression", the word "literal" always refers to a piece of the source code, rather than some value your program manipulates at runtime.

Strings in quotes are also literals. In `std::cout << "Hello!\n";`, the `"Hello!\n"` is a **string literal**.

## Exercise 3

Take one of the previous examples and find every literal in it.
