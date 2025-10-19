# Vectors

The plain arrays are limited in many ways. One of them, as explained before, is the fact that their size must be fixed at compile-time.

The solution to that is...

## `std::vector`

`#include <vector>` grants you access to `std::vector`.

For most purposes it behaves like an array, but its size can be determined at runtime, and can be changed after the vector is created.

This is how you use a `vector`:

Array | Vector | Comment
---|---|---
`int arr[10] = {};`|`std::vector<int> arr(10);`|Creating an array of `10` zeroes.
`int arr[10];`|—|There's no (simple) way of creating a vector with uninitialized elements.
`int arr[] = {1,2,3};`|`std::vector<int> arr = {1,2,3};`
~~`int arr[0];`~~|`std::vector<int> arr;`| Arrays can't be empty but vectors can. This makes sense for them becase you can add elements to them later.<br/>Vectors are empty by default (can't be uninitialized).
`arr[i]`|`arr[i]`|Accessing the elements works the same way.

Notice that creating a vector of a specific size is done using `std::vector<int> arr(10);`, not `[10]`. Trying to use `[10]` would give you an array of empty vectors.

Now the unique features of vectors:
* **`arr.size()`** gives you the current vector size.

  For example:
  ```cpp
  std::vector<int> v = {1, 2, 3};
  std::cout << v.size() << "\n"; // 3
  ```

* **`arr.empty()`** results in `true` if the vector is empty and `false` otherwise, it is a shorthand for `arr.size() == 0`.

  For example:
  ```cpp
  std::vector<int> a;
  std::vector<int> b = {10, 20, 30};
  if (!a.empty())
      std::cout << a[0] << "\n"; // Not executed, the condition is false.
  if (!b.empty())
      std::cout << b[0] << "\n"; // 10
  ```

* **`arr.push_back(42);`** adds an element to the end of the vector.

  For example:
  ```cpp
  std::vector<int> a;
  std::cout << a.size() << "\n"; // 0
  a.push_back(10); // After this the vector holds `{10}`.
  std::cout << a.size() << "\n"; // 1
  a.push_back(20); // After this the vector holds `{10, 20}`.
  std::cout << a.size() << "\n"; // 2
  a.push_back(30); // After this the vector holds `{10, 20, 30}`.
  std::cout << a.size() << "\n"; // 3
  std::cout << a[0] << " " << a[1] << " " << a[2] << "\n"; // 10 20 30
  ```

  Note that accessing a vector using `[]` out of bounds doesn't automatically insert the element. Like for arrays, this is UB.

  Note that vectors always stores copies of the elements. Modifying a variable after inserting it doesn't change the vector contents.

* **`arr.pop_back();`** removes the last element.

  It is UB to call this on an empty vector.

  ```cpp
  std::vector<int> a = {10, 20, 30};
  a.pop_back(); // After this the vector holds `{10, 20}`.
  ```

* **`arr.clear();`** removes all elements. Does nothing if the vector is already empty.
  ```cpp
  std::vector<int> a = {10, 20, 30};
  std::cout << a.size() << "\n"; // 3
  a.clear();
  std::cout << a.size() << "\n"; // 0
  ```

* **`arr.resize(3);`** changes the vector size . If it's larger than before, the new elements are zeroed. If it's smaller than before, the excess elements are removed.

  ```cpp
  std::vector<int> a = {10, 20, 30};
  a.resize(5); // `a` becomes `{10, 20, 30, 0, 0}`.
  a.resize(2); // `a` becomes `{10, 20}`.
  ```



* Also, you can assign to an entire vector at once, including from another vector. You can also initialize one vector with another.

  ```cpp
  std::vector<int> a = {1,2,3};
  std::vector<int> b = a; // b = {1,2,3}
  a = b;
  b = {1,2,3};
  ```
  Compare to arrays:
  ```cpp
  int a[] = {1,2,3};
  int b[] = a; // Compilation error.
  a = b;       // Compilation error.
  b = {1,2,3}; // Compilation error.
  ```

### A practical example

Let's practice a bit:
```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v;

    while (true)
    {
        int x = 0;
        std::cin >> x;
        if (x == 0)
            break;
        v.push_back(x);
    }

    for (int i = 0; i < v.size(); i++)
        std::cout << v[i] << "\n";
}
```
This lets you input any amount of numbers, until you input `0`. Then it prints back all those numbers.

If you input `1 2 3 0`, this prints `1 2 3`.

Also note that I'm doing `std::vector<int> v;` here, despite the earlier advice to not leave variables uninitialized. As I said above, vectors always default to empty, so there is no harm in doing this. Out of the types you already learned, only numbers (which includes `bool`) and arrays of those can be truly uninitialized and should be manually initialized.

(Advanced readers might be yelling at their screens now for me not using `size_t` here. Yes, I know. This will be explained later.)

> ## Exercise 1
>
> Write a program that lets the user input as many integers as they like, ending with `0`.
>
> Add the positive numbers to one vector and the negative numbers to another. After the user inputs `0`, print both vectors.

## Multidimensional vectors

`std::vector<std::vector<int>> v;` is a vector of vectors.

Unlike multidimensional arrays, here each inner vector can have different size (this is what's called a "jagged" array).

If you don't need the jaggedness (if you just need a rectangular array), it's recommended to avoid vectors of vectors, and instead use a longer one-dimensional vector, as that should be faster:

Compare:
```cpp
std::vector<std::vector<int>> v = {
    {1, 2, 3},
    {4, 5, 6},
};

for (int y = 0; y < v.size(); y++)
{
    for (int x = 0; x < v[y].size(); x++)
        std::cout << " " << v[y][x];

    std::cout << "\n";
}
```
```cpp
std::vector<int> v = {
    1, 2, 3, // Spelling the numbers on different lines doesn't change anything.
    4, 5, 6,
};

for (int y = 0; y < 2; y++)
{
    for (int x = 0; x < 3; x++)
        std::cout << " " << v[y * 3 + x];

    std::cout << "\n";
}
```

> ## Exercise 2
>
> Make a vector of vectors, and fill it with numbers in a jagged manner (so that each inner vector has different size). Print it using a nested loop, so that each inner vector is on its own line. Observe that different lines show different number of elements, so the vector is indeed jagged.

## Comparing vectors

`==` and `!=` work on vectors as you would expect:

```cpp
std::vector<int> a = {1,2,3}, b = {1,2,3}, c = {4,5,6}, d = {7,8};
std::cout << (a == a) << '\n'; // 1 (equal)
std::cout << (a == b) << '\n'; // 1 (equal)
std::cout << (a == c) << '\n'; // 0 (not equal)
std::cout << (a == d) << '\n'; // 0 (not equal)
```
If the elements (or the number of them) are differet, the vectors are considered to be not equal.

`<`,`<=`,`>`,`>=` work too, but their behavior is less intuitive. They compare the vectors **lexicographically**, which literally means "in dictionary order", i.e. `a < b` on vectors means that `a` would come before `b` in a dictionary (but comparing vector elements instead of letters).

First, `a[0]` is compared with `b[0]`. If not equal, this is the result of the comparison. Otherwise `a[1]` is compared with `b[1]`, and so on. If one vector runs out of elements before the other, the shorter vector is considered less than the longer (but this only happens )

`a`|&nbsp;|`b`|Comment
---|---|---|---
`{10,20,30}`|`==`|`{10,20,30}`|
`{ 9,20,30}`|`<`|`{10,20,30}`|
`{11,20,30}`|`>`|`{10,20,30}`|
`{10,19,30}`|`<`|`{10,20,30}`|
`{11,19,30}`|`>`|`{10,20,30}`|Since the first element is smaller, the remaining elements are not compared.
`{10,20}`|`<`|`{10,20,30}`|
`{11,20}`|`>`|`{10,20,30}`|Since there are unequal elements, it doesn't matter which vector is shorter.
