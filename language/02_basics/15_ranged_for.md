# Range-based `for` loop

There's another variant of the `for` loop, called **range-based `for`** or just **ranged `for`**:

```cpp
int arr[] = {10, 20, 30};

for (int elem : arr)
{
    std::cout << elem << "\n";
}
```
This prints all elements of the array.

Here `int` in `int elem` should match the array type.

Ranged `for` doesn't keep track of the element indices. If you need to know the index, keep track of it manually using another variable, or just don't use a ranged `for`.

Ranged `for` works with both arrays and vectors (and some other things). It is said to "iterate over" arrays/vector/etc.

Braces are optional in ranged `for`, like in the normal one.

The example above is mostly equivalent to:
```cpp
for (int i = 0; i < 5; i++)
{
    std::cout << arr[i] << '\n';
}
```
Note that it we had an array of `double`, the first example would use `double elem`, while the second one would still use `int i`, as the indices are still integers.

> ## Exercise 1
>
> Rewrite one of your previous programs to use a ranged `for` loop.

## Modifying the elements

```cpp
int arr[] = {10, 20, 30};

for (int elem : arr)
    elem = 0;

for (int elem : arr)
    std::cout << elem << "\n";
```
(I'm omitting the braces for brevity.)

What do you think this prints? Surprisingly, this prints `1 2 3`.

The reason for this is that `int elem` is a **copy** of the respective element, changing which doesn't affect the original.

This is very similar to the following:
```cpp
int x = 10;
int y = x;
y = 20;
std::cout << x << "\n"; // Still 10.
```

The fix for that is:
```cpp
for (int &elem : arr)
    elem = 0;
```
`&` makes `elem` a **reference**, as opposed to a regular variable.

References will be explain in more detail later, but roughly speaking, references act as other names for variables (or rather for *objects*, since the individual array elements are not variables, only the whole array is; the correct word that covers (most) variables, array elements, etc is "object").

Here's a simpler (but not very practical) example demonstrating the use of references:
```cpp
int x = 10;
int &y = x;
y = 20;
std::cout << x << "\n"; // This is now 20.
```

> ## Exercise 2
>
> Make a program that lets the user input as many numbers as they like (stop at `0`), then increases each of them by 10 and prints them.
>
> Use a `std::vector` to store the numbers. Use three loops: the second one should be a ranged `for` with a reference to modify the numbers, and the third one a ranged `for` without a reference to print them. Try removing the reference from the second loop and see what happens.
>
> Now merge the second and the third loop. Try using a reference in it and not using one, see what happens.
>
> Now rewrite this program to use a single loop and no vectors, by immediately printing each modified number after inputting it. Observe the difference in how it handles the input (hint: try inputting all numbers on one line, and on different lines).

## Modifying a vector while iterating over it

You're not allowed to add or remove elements to a vector while iterating over it with ranged-for. This causes undefined behavior, and the reasons for this will be discussed in future chapters.

If you need to do this, use the regular `for` loop.

> ## Exercise 3
>
> Write a program that loops over a vector, printing each element, and if it's larger than 10, adds that number minus 10 to the end of the vector (and the new numbers should be handled in the same manner in the same loop too).
>
> Try doing this with ranged-for and observe the breakage. Then try this with the regular `for` loop.
