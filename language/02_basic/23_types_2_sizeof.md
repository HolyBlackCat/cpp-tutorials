# Sizes of types

Each [object](/language/02_basic/20_objects_and_temporaries.md#variables-and-values) occupies a certain amount of memory (normally RAM), depending on its type.

You can check this amount using the **`sizeof`** operator. For example, `sizeof(int)` is typically `4`, meaning a single `int` occupies 4 bytes.

Check the sizes of other types you know. Typically, `sizeof(double) == 8`, `sizeof(bool) == 1`, and `sizeof(char) == 1` (the last one is guaranteed).

Those sizes matter, e.g. because if you have a lot of objects (such as in a large array), you need to ensure that they fit into your RAM.

## Expressions

`sizeof` can be applied to [expressions](/language/02_basic/03_expressions.md#expressions) too. For example, `sizeof 42` has the same meaning as `sizeof(int)`, since `42` is an `int`.

`sizeof(42)` has the same meaning as `sizeof 42`, here `(...)` can be omitted for expressions. It can't be omitted for types though, `sizeof int` is a compilation error.

The expression operand of `sizeof` is "unevaluated", meaning that any functions in it are not called, assignments are not executed, etc. For example:

```cpp
int foo()
{
    std::cout << "Hello from foo!\n";
    return 42;
}

int main()
{
    // Typically prints 4, since it acts on the return type of `foo()`, which is `int`.
    // Doesn't print `Hello from foo!`, the function is not actually called.
    std::cout << sizeof foo() << '\n';
}
```

## Arrays

For arrays, the size of the element type is multiplied by the array size. E.g. given `int arr[10];`, `sizeof arr` (aka `sizeof(int[10])`) is equal to `sizeof(int) * 10`, which is typically 40.

You can confirm this in the task manager. Create a large array, fill it with some values, and observe memory usage:
```cpp
#include <iostream>

const int size = 32 * 1024 * 1024;
int arr[size];

int main()
{
    for (int i = 0; i < size; i++)
        arr[i] = i;

    int blah;
    std::cin >> blah; // Pause the program to let you observe the size.
}
```

This should use ~128 MB of memory: `sizeof(int) * 32` is `128` bytes, then `128 * 1024` is 128 kilobytes, and `128 * 1024 * 1024` is 128 megabytes. (Whether 1 KB is 1024 bytes or 1000 bytes)

Notice that I'm creating a [global](/language/02_basic/19_functions_1.md#global-variables) array. The total combined size of [local](/language/02_basic/19_functions_1.md#global-variables) variables is usually limited by 1 megabyte or a few megabytes (causing a runtime error if exceeded), so it's harder to observe memory usage with local arrays. This will be explained in more detail later. The contents of `vector`s and `string`s don't count against this limit.

Notice that I'm filling the array with some (non-zero) numbers. Modern OSes might not actually put the array in memory otherwise, so it wouldn't show up in the task manager.

C people sometimes use `sizeof arr / sizeof arr[0]` to determine the number of elements in an array, but in C++ you should prefer `std::size(arr)`, since this works correctly for strings and vectors too, while the `sizeof` trick doesn't ([see below](#containers)).

## Structs

`sizeof` of a `struct` typically equals to the sum of `sizeof` of its fields, or sometimes a bit more (for reasons that will be explained later).

## References

`sizeof` ignores reference-ness, so e.g. `sizeof(int &) == sizeof(int)`.

This doesn't mean that it's the actual size of the reference, though. To determine its true size, put it in a struct and `sizeof` that:
```cpp
struct A
{
    int &x;
};

struct B
{
    char &x;
};

std::cout << sizeof(A) << '\n'; // Typically 8.
std::cout << sizeof(B) << '\n'; // Typically 8.
```
Typically all references occupy the same size, 8, for reasons that will become apparent later.

This is also one of the reasons why you shouldn't try to use references to create alternative names for fields, since that creates memory overhead:

```cpp
struct A
{
    int color;
    int &colour = color; // Bad, the reference wastes memory.
};
```

## Containers

If you try applying `sizeof` to `std::string` or `std::vector`, you'll quickly realize that it always returns a fixed number that doesn't depend on the number of elements in them (for reasons that, again, will become apparent later).

You can get the combined size of the elements e.g. this way:
```cpp
std::vector<int> v = {1, 2, 3};

std::cout << v.size() * sizeof(int) << '\n';
std::cout << v.size() * sizeof(v[0]) << '\n'; // This works too. Even if the vector is empty, because `v[0]` is not evaluated, see above.
```

The `sizeof(std::vector<int>)` is the memory overhead of the vector itself, it uses this much memory for its internal bookkeeping. (The situation with `std::string` is a bit more complicated, but that's for later.)

## `sizeof` is a compile-time constant

`sizeof` is fixed at compile-time (and so is `CHAR_BIT`).

This means it's constant enough to be used as array size: `char blah[sizeof(int)];` is valid.

It being a compile-time constant is one of the reasons why `sizeof` of a `vector` can't possibly reflect the number of elements in a vector, since that can vary at runtime.

If your compiler is not correctly configured to [reject variable-length arrays](/language/02_basic/11_arrays.md#array-size), you may observe `sizeof` of those arrays changing at runtime. But neither variable-length arrays nor non-constant `sizeof` are legal in standard C++.

## `void`

`sizeof(void)` doesn't compile, which is consistent with `void x;` not compiling.

In general, `sizeof` always returns `1` or more.

## `bool`
