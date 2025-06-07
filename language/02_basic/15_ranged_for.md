# Range-based `for` loop

There's another variant of the `for` loop, called **range-based `for`** or just **ranged `for`**:

```cpp
double arr[] = {1.5, 2.5, 3.5};

for (int elem : arr)
{
    std::cout << elem << "\n";
}
```
This prints all elements of the array.

Here `double` in `double elem` should match the array element type.

Ranged `for` doesn't keep track of the element indices. If you need to know the index, keep track of it manually using another variable, or just don't use a ranged `for`.

Ranged `for` works with both arrays and vectors (and some other things). It is said to "iterate over" the array/vector/etc.

Braces are optional in ranged `for`, like in the normal `for`.

The example above is mostly equivalent to:
```cpp
for (int i = 0; i < 5; i++)
{
    std::cout << arr[i] << "\n";
}
```
Notice that we're using `int` instead of `double` here, because even though the array elements are `double`s here, the indices are still integers.

> ## Exercise 1
>
> Rewrite one of your previous programs to use a ranged `for` loop.


## Modifying a vector while iterating over it

A word of warning.

You're not allowed to add or remove elements to a vector while iterating over it with a ranged-for. This causes undefined behavior, and the reasons for this will be discussed in future chapters.

If you need to do this, use the regular `for` loop with indices.
