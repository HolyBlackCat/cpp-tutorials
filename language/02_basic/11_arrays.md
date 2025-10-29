# Arrays

Arrays are variables that hold more than one value.

```cpp
int arr[3];
std::cin >> arr[0];
std::cin >> arr[1];
std::cin >> arr[2];

std::cout << "First:  " << arr[0] << "\n";
std::cout << "Second: " << arr[1] << "\n";
std::cout << "Third:  " << arr[2] << "\n";
```

The `3` in `int arr[3];` (the **size** of the array) means that it can hold 3 values, which can then be accessed as `arr[0]`, `arr[1]`, `arr[2]` (`0`,`1`,`2` being the **indices**). Note that the indices start from zero.

Note that accessing `arr[3]` (same index as the array size) and larger indices is illegal (`arr[3]` would be the fourth element, but this array only has three).

While in this example the array doesn't look very useful compared to `int arr0, arr1, arr2;`, things change when you start using loops. This is explained later in this chapter.

## The meaning of `[N]`

Notice that the meaning of `[N]` depends on where it is used:

* When used in a declaration, it sets the array **size**.

* When used used outside of a declaration, it refers to **one of the array elements**.

## Initialization

Like other variables, arrays are uninitialized by default.

You can initialize them:
```cpp
int arr[3] = {10, 20, 30};
```
If you initialize the array, you can omit the size:
```cpp
int arr[] = {10, 20, 30};
```
This is exactly the same as the above.

But if you do specify the size and provide **more** initializers (elements) than the size, this is a compilation error.

If you provide **less** elements than the size, the remaining elements are zeroed:
```cpp
int arr[5] = {10, 20, 30};
```
This is exactly the same as `= {10, 20, 30, 0, 0}`.

Therefore `= {}` is a common way to initialize an array to all zeroes.

## Assignment

In the first example you saw that `std::cin >> arr[0];` is valid, so as you might have expected, you can assign to the elements too:

```cpp
arr[0] = 42;
```
This is legal, despite `arr[0]` not being a variable (only the entire `arr` is technically called a "variable" here).

So the lhs of `=` isn't always a variable. A more general rule will be explained in future chapters.

You can't assign the entire array at once.
```cpp
int arr[3];
arr = {10, 20, 30}; // Compilation error.
```
You also can't print the entire array at once, and can't input the entire array with a single use of `std::cin >>`. Only the individual elements. There is no technical reason for this, this is simply how `std::cout` and std::cin` are designed.

## Loops

While the size must be a compile-time constant, the indices don't. In fact, you'll often see loop counters (counter variables) as indices:

```cpp
int arr[5];

for (int i = 0; i < 5; i++)
    std::cin >> arr[i];

for (int i = 0; i < 5; i++)
    std::cout << arr[i] << "\n";
```
This reads 5 values, and then prints all of them.

> ## Exercise 1
>
> Write a program that lets you input 10 numbers, and then prints their sum.
>
> Use an array of size 10, one loop to input, then another to sum.
>
> Now rewrite this program to not use an array, and do the summation in the same single loop as the input.

## Array size

The array size must be larger than `0`.

Once an array is created, its size can't be changed.

And worse, the array size must be fixed at compile-time. It can't depend on the user input:
```cpp
int size;
std::cin >> size;
int arr[size]; // Compilation error.
```
Try it.

It is possible that this will actually compile for you, but this is a sign of an improperly configured compiler. Compilers tend to accept some invalid things by default (that are disallowed by the C++ standard, the document that describes how C++ is supposed to work), and need to be configured not to. If you're also following my tooling tutorial, it [explains this in more detail](/tooling/articles/recommended_compiler_flags.md). If you're using GCC or Clang, use `-std=c++26 -pedantic-errors` (or some other `-std=c++??`) to make the compiler error on this. MSVC rejects this by default (but you should still use `/std:c++latest` to make it reject other non-standard things).

You might be wondering, if this happens to work for you, why do anything? Isn't this a useful feature to have?

First of all, you should try to write code that works on all three big compilers, and this doesn't work on one of the three (MSVC). Second, VLAs (variable-length arrays, which is how this feature is called) seem to be [bug-prone in general](https://stackoverflow.com/q/12407754/2752075). C++ has better alternatives (namely `std::vector`), which will be discussed in the later chapters.

## Multi-dimensional arrays

`int arr[3];` is said to be one-dimentional (1D).

But arrays can also be multi-dimensional:
```cpp
int arr[3][5];
```
This is a 2D array, a rectangular table of numbers of size 10×20. Larger dimensions are possible too: `int arr[3][5][7];` and so on.

You can think of `int arr[3][5];` as an array of arrays. It's an array of size `3`, the elements of which are themselves arrays (of 5 `int`s each).

They can be initialized like so:
```cpp
int arr[3][5] = {
    {1, 2, 3, 4, 5},
    {6, 7, 8, 9, 10},
    {11, 12, 13, 14, 15},
};
```

And to access the elements, you do `arr[1][2] = 42;`, as you would expect.

Notice the comma after the last element. It does nothing, it's allowed but not required. It's always allowed, even in `int arr[] = {1, 2, 3,};`. It's a good idea to include the comma when you spell the initializers on multiple lines, as it makes it easier to add/remove them and move them around.

### Rows vs columns

People naturally tend to ask which index is which, is it `arr[row][column]` or `arr[column][row]`.

The answer is: whatever you decide. The language itself doesn't give an inherent meaning to the indices.

Though by convention people usually prefer `arr[row][column]`.

> ## Exercise 2
>
> Write a program that uses nested loops to fill an array with those numbers:
>
> ```
>  1  2  3  4  5
>  6  7  8  9 10
> 11 12 13 14 15
> ```
> Do not use an array initializer to achieve this (so not like in the last example).
>
> Then use another pair of nested loops to print the array.

## Declaring several arrays in one declaration

You might've guessed this by now, but to declare multiple arrays in one declaration, the size has to be repeated for each of the arrays:

```cpp
int a[3], b[4]; // ok, both are arrays
int a, b[4]; // ok, only `b` is an array
int a[3], b; // ok, only `a` is an array
int[3] a, b; // compilation error, `[...]` must be after the variable name
int[3] a; // compilation error, `[...]` must be after the variable name
```

The proper terminology for this is relatively little-known:

* The part of a declaration shared between all variables is called the **decl-specifier-seq**.
* The individual variable names with `[...]` if any are the **declarators**.

E.g. in `int a[3], b;`, `int` is the decl-specifier seq, and `a[3]` and `b` are the declarators.
