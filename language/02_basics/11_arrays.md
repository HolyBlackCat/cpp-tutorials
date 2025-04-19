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

Note that `arr[3]` and larger is illegal (`arr[3]` would be the fourth number).

So far this probably doesn't look too useful compared to just `int arr0, arr1, arr2;`, but bear with me. It'll make sense in the next chapter.

## Initialization

Like other variables, arrays are uninitialized by default.

You can initialize them:
```cpp
int arr[3] = {10, 20, 30};
```
In this can you can omit the same, this is completely equivalent:
```cpp
int arr[] = {10, 20, 30};
```
If you specify the size and provide less initializers than the size, the remaining elements are zeroed:
```cpp
int arr[5] = {10, 20, 30};
```
This is exactly the same as `= {10, 20, 30, 0, 0}`.

And if you provide **more** elements than the size, this is a compilation error.


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
You also can't print the entire array at once, and can't input the entire array with a single use of `std::cin >>`.

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

It is possible that this actually will compile for you, but this is a sign of an improperly configured compiler. Compilers tend to accept some invalid things by default, and need to be configured not to. If you're also following my tooling tutorial, this is a good time to read about [the recommended compiler settings](/tooling/articles/recommended_compiler_flags.md). If you're using GCC or Clang, use `-std=c++26 -pedantic-errors` (or some other `-std=c++??`) to make the compiler error on this. MSVC should reject this by default.

You might be wondering, if this happens to work for you, why do anything? First of all, you should try to write code that works on all three big compilers, and this doesn't work on one of the three (MSVC). Second, VLAs (variable-length arrays, which is how this is called) seem to be [bug-prone in general](https://stackoverflow.com/q/12407754/2752075). C++ has better alternatives (namely `std::vector`), which will be discussed in the later chapters.

## Loops

While the size must be a compile-time constant, the indices don't. In fact, you'll often use a loop variable as the index:

```cpp
int arr[5];

for (int i = 0; i < 5; i++)
    std::cin >> arr[i];

for (int i = 0; i < 5; i++)
    std::cout << arr[i] << "\n";
```
This reads 5 values, and then prints all of them.

## Multi-dimensional arrays.

`int arr[3];` is said to be one-dimentional (1D).

But arrays can also be multi-dimensional:
```cpp
int arr[3][5];
```
This is a 2D array, a rectangular table of numbers of size 10×20. Larger dimensions are possible too: `int arr[3][5][7];` and so on.

You can think of `int arr[3][5];` as an array of arrays. It's an array of size `3`, the elemenets of which are themselves arrays (of 5 `int`s each).

And to access the elements, you do `arr[1][2] = 42;`, as you would expect.

### Rows vs columns

People naturally tend to ask which index is which, is it `arr[row][column]` or `arr[column][row]`.

The answer is: whatever you decide. The language itself doesn't give an inherent meaning to the indices.

Though by convention people usually prefer `arr[row][column]`.
