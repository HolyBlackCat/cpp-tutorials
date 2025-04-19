# Vectors

The plain arrays are limited in many ways. One of them, as explained before, is the fact that their size must be fixed at compile-time.

The solution to that is...

## `std::vector`

`#include <vector>` grants you access to `std::vector`.

For most purposes it behaves like an array, but its size can be determined at runtime, and can be changed after the vector is created.

This is how you use a `vector`:

Array | Vector | Comment
---|---|---
`int arr[10];`|`std::vector<int> arr(10);`|The vector zeroes all elements by default, unlike the array.
`int arr[] = {1,2,3};`|`std::vector<int> arr = {1,2,3};`
~~`int arr[0];`~~|`std::vector<int> arr;`| Arrays can't be empty but vectors can. This makes sense for them becase you can add elements to them later.
`arr[i]`|`arr[i]`|Accessing the elements works the same way.

Notice that creating a vector of a specific size is done using `std::vector<int> arr(10);`, not `[10]`. Trying to use `[10]` would give you an array of empty vectors.

Now the unique features of vectors:
* `arr.size()` gives you the current vector size.
* `arr.push_back(42);` adds an element to the end of the vector.
* `arr.pop_back();` removes the last element.
* `arr.resize(3);` changes the vector size (to `3` here). If it's larger than before, the new elements are zeroed. If it's smaller than before, the excess elements are removed.

* Also, you can assign to an entire vector at once, including from another vector. You can also initialize one vector with another.

  ```cpp
  std::vector<int> a = {1,2,3};
  std::vector<int> b = a;
  a = b;
  b = {1,2,3};
  ```
  This is in contrast with arrays, which don't support any of that:
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
        int x;
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

(Advanced readers might be yelling at their screens now for me not using `size_t` here. Yes, I know. This will be explained later.)

## Multidimensional vectors

`std::vector<std::vector<int>> v;` is a vector of vectors.

Unlike multidimensional arrays, here each inner vector can have different size (this is what's called a "jagged" array).
