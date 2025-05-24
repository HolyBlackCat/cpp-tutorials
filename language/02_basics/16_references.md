# References

If you experiment with the ranged-for a bit, you'll quickly realize that any changes you make to the elements are seemingly not reflected in the original array or vector (at least by default).

Have a look:

```cpp
int arr[] = {10, 20, 30};

for (int elem : arr)
{
    elem += 100;
    std::cout << elem << "\n"; // 110 120 130
}

for (int elem : arr)
    std::cout << elem << "\n"; // 10 20 30, somehow!
```
`int elem` is a copy of the array element, that exists until the end of the current iteration. So this is the same issue as:
```cpp
int a = 10;
int b = a;
b = 20; // Or `b += something;`, same thing.
std::cout << a << "\n"; // Still 10!
```

The solution to this is...

## References

```cpp
int arr[] = {10, 20, 30};

for (int &elem : arr) // Note the `&` here!
{
    elem += 100;
    std::cout << elem << "\n"; // 110 120 130
}

for (int elem : arr)
    std::cout << elem << "\n"; // 110 120 130
```

References can be created outside of loops too:
```cpp
int a = 10;
int &b = a;
b = 20;
std::cout << a << "\n"; // 20
```

Roughly speaking, a reference is a special kind of variable that act as an "alternative name" or "alias" for another thing.

It has to be initialized when created, and then can't be modified to refer to something else (because assigning to it modifies the thing it refers too, instead of making it refer to something else):
```cpp
int &a; // Error, references must be initialized.
```
```cpp
int a = 10;
int &b = a;
int c = 11;
b = c; // This modifies `a`, rather than making `b` refer to `c`.
b = 42; // This modifies `a`
std::cout << a << "\n"; // 42
std::cout << c << "\n"; // 11
```

> ## Exercise 1
>
> Experiment with references. Write a program that modifies the array elements in ranged-for, observe that it doesn't work correctly without references, and does work correctly with them.

## References to other references

```cpp
int x;
int &y = x;
int &z = y;
```
Here `int &z = y;` is entirely equivalent to `int &z = x;`.

In other words, a reference can't actually refer to another reference. Instead, trying to do so makes it refer to whatever that reference refers to.

## Const references

Perhaps unintuitively, `const` references do exist:

```cpp
for (const int &elem : arr)
    std::cout << elem << "\n";
```
Here `elem` is a const reference. Or more correctly, a "reference to `const` `int`".

Here `elem` can't be modified (because the reference is const), so you might ask what's the point, why not simply `int elem`?

In this case it's indeed pointless, because `int` is so cheap to copy, but imagine that you have a `std::vector<std::vector<int>>`, where each inner vector has a few million elements.

Trying to iterate over it using `for (std::vector<int> a : b)` would temporarily copy each inner vector, which is unnecessarily slow, and is a waste of memory. It would be much better to use `for (const std::vector<int> &a : b)` to avoid the copying.

Now, why `const std::vector<int> &a` instead of just `std::vector<int> &a`? First of all, if `b` is const, a non-const reference wouldn't compile, because otherwise that would be a loophole for modifying const variables:

```cpp
const int x = 42;
int &y = x; // Compilation error!
y = 43; // Because otherwise this would allow you to modify a const variable.
```

And second of all, even if `b` is non-const, you might want to make the `a` reference const to prevent yourself from accidentally modifying it.

> ## Exercise 2
>
> Create a large vector of vectors and try to observe the difference in performance between references and copies.

## What else can references refer to?

Array and vector elements, for example.

You'll often see things like this, if for some reason the ranged-for can't be used:
```cpp
std::vector<double> vec = {/*...*/};
for (int i = 0; i < vec.size(); i++)
{
    double &elem = vec[i]; // To avoid spelling `vec[i]` every time.
    // Do something to `elem`.
}
```

## Dangling references

If the object that a reference points to is destroyed, using the reference after that causes undefined behavior:

```cpp
std::vector<int> vec = {1, 2, 3};
int &ref = vec[0];
vec.clear(); // Remove all elements from the vector.
ref = 42; // UB!
std::cout << ref << '\n'; // Also UB!
```
A reference that points to an object that no longer exists is called a **dangling reference**.

Perhaps unintuitively, the same can happen when *growing* the vector too.
```cpp
std::vector<int> vec = {1, 2, 3};
int &ref = vec[0];
vec.push_back(4);
ref = 42; // This is UB too!
std::cout << ref << '\n'; // Also UB!
```
The reasons for this will be explained later.

> ## Exercise 3
>
> Try to observe breakage due to dangling references in those two cases.

## More corner cases

There are also some weird corner cases like:

```cpp
int &x = 10; // Compilation error.
const int &y = 10; // Works.
```

...which aren't very useful right now, and will be explained in more detail later.
