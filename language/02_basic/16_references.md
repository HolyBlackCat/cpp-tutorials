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

Or a simple example without loops:
```cpp
int a = 10;
int &b = a;
b = 20;
std::cout << a << "\n"; // 20
```

Roughly speaking, a reference is a special kind of variable that act as an "alternative name" or "alias" for another thing.

It has to be initialized (when declared), and then can't be modified to refer to something else (because assigning to it modifies the thing it refers too, instead of making it refer to something else):
```cpp
int &a; // Compilation error, references must be initialized.
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

## The meaning of `=` and `&`

Notice that `=` has different effect in a reference declaratation vs when assigning to a reference (which is what happens outside of declarations):
```cpp
int x = 42;
int &a = x; // `=` sets the target for the reference
a = 43; // `=` changes the value of the target
```

Similarly, `&` creates a reference only when used in declarations. Elsewhere it has an entirely different meaning, which we'll cover later.

> ## Exercise 1
>
> Experiment with references. Write a program that modifies the array elements in ranged-for, observe that it doesn't work correctly without references, and does work correctly with them.

## Declaring several references in one declaration

If you declare several variables in one declaration, `&` will only affect one of them at a time. This is the same situation [as with arrays](./11_arrays.md#declaring-several-arrays-in-one-declaration).

E.g. given `int x = 42;`...

```cpp
int &a = x, &b = x; // both are references
int a = x, &b = x; // only `b` is a reference
int &a = x, b = x; // only `a` is a reference
int& a = x, b = x; // only `a` is a reference
```
Notice how the whitespace around `&` doesn't affect to what it applies, since [C++ in general isn't sensitive to whitespace](./01_your_first_program.md#whitespace-and-line-breaks).

## References to other references

```cpp
int x;
int &y = x;
int &z = y;
```
Here `int &z = y;` is entirely equivalent to `int &z = x;`.

In other words, a reference can't actually refer to another reference. Instead, trying to do so makes it refer directly to whatever that reference refers to.

## Const references

Perhaps unintuitively, `const` references do exist:

```cpp
for (const int &elem : arr)
    std::cout << elem << "\n";
```
Here `elem` is a const reference. Or more correctly, a "reference to `const` `int`" (references can never be actually const themselves, but the phrase "const reference" is commonly used to mean a reference to something const; this tutorial uses the phrase "const reference" a lot, and I don't see a point in being pedantic with this).

Here `elem` can't be modified (because the reference is const), so you might ask what's the point, why not simply `int elem`?

In this case it's indeed pointless, because `int` is so cheap to copy, but imagine that you have a `std::vector<std::vector<int>>`, where each inner vector has a few million elements.

Trying to iterate over it using `for (std::vector<int> a : b)` would temporarily copy each inner vector, which is unnecessarily slow, and is a waste of memory. It would be much better to use `for (const std::vector<int> &a : b)` to avoid the copying.

### Why not always use non-const references?

Now, why `const std::vector<int> &a` instead of just `std::vector<int> &a`? First of all, if `b` is const, a non-const reference wouldn't compile, because otherwise that would be a loophole for modifying const variables:

```cpp
const int x = 42;
int &y = x; // Compilation error!
y = 43; // Because otherwise this would allow you to modify a const variable.
```

And secondly, even if `b` is non-const, you might want to make the `a` reference const to prevent yourself from accidentally modifying the element.

### Mixing constness

As is hopefully clear by now, const references can refer to non-const variables, but attempting the opposite is a compilation error:

```cpp
int a = 10;
const int b = 20;

int &r1 = a; // ok
int &r2 = b; // compilation error
const int &r3 = a; // ok
const int &r4 = b; // ok
```
Similarly:
```cpp
std::vector<int> a = {1, 2, 3};
const std::vector<int> b = {1, 2, 3};

for (int &r : a) {} // ok
for (int &r : b) {} // compilation error
for (const int &r : a) {} // ok
for (const int &r : b) {} // ok
```

Notice also that you can't initialize a non-const reference with a const **reference**, even if that points to a non-const variable:

```cpp
int a = 10;
const int &b = a;
int &c = b; // Compilation error.
```
Firstly, if this wasn't the case, this would be a loophole to remove constness from a reference. And secondly, whether or not this compiles is determined at compile-time (obviously), while whether or not `a` points to a `const` variable can't be known until runtime.

### Modifying the target of a const reference

```cpp
int a = 10;
const int &b = a;
// b = 20; // Compilation error.
a = 20; // ok
std::cout << a << "\n"; // 20
std::cout << b << "\n"; // 20
```
While you can't assign to a const reference, you can modify it's target directly, without using the reference.

This is legal, and accessing the reference after that will correctly give the updated value.

> ## Exercise 2
>
> Create a large vector of vectors and try to observe the difference in performance between references and copies.

## What else can references refer to?

In case it's not clear, you can manually create references to array/vector elements.

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
Growing a vector can invalidate (make dangling) the references to its elements. The reasons for this will be explained later.

> ## Exercise 3
>
> Try to observe breakage due to dangling references in those two cases.

## References and arrays

It is an error to create an array or vector of references:

```cpp
int x, y, z;

int &arr[3] = {x, y, z}; // error
std::vector<int &> vec = {x, y, z}; // error
```

But references to entire arrays/vectors can exist:
```cpp
int arr[3] = {10,20,30};
int (&r1)[3] = arr; // Reference to array.
std::cout << r1[1] << '\n'; // 20

std::vector<int> vec = {10,20,30};
std::vector<int> &r2 = vec; // reference to vector.
std::cout << r2[1] << '\n'; // 20
```
Notice `int &arr[3]` vs `int (&arr)[3]` - an array of references vs a reference to an array. This will be explained in more details in later chapters.

## More corner cases

There are also some weird corner cases like:

```cpp
int &x = 10; // Compilation error.
const int &y = 10; // Works.
```

...which aren't very useful right now, and will be explained in more detail later.
