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
