# `const` parameters and return types

What if you make a parameter or a return type of a function `const`?

Note that I'm not talking about "const references", since that is just an informal/inaccurate way of saying "references to const", they are not const themselves.

I'm talking about e.g. `const int`.

### `const` parameters

When used on a parameter, it has zero effect on the caller, and just prevents the function from modifying its parameter (which is a copy, so the change wouldn't affect the caller anyway). This can only make sense to prevent yourself from accidentally modifying the parameter when you don't intend to. Or to signal to whoever reads the code that the parameter isn't modified. For example:

```cpp
void foo(int a, const int b)
{
    a = 100;
    b = 200; // This line doesn't compile, because `b` is const.
}

int main()
{
    // The exact same things can be passed to both parameters:

    foo(10, 20); // Ok.

    int x = 10, y = 20;
    foo(x, y); // Ok.

    std::cout << x << ' ' << y << '\n'; // Still prints `10 20`, the caller doesn't see the change to `x`.
}
```

Also making a parameter `const` still doesn't allow you to [use it as an array size](13_constants.md#compile-time-constants), regardless of what argument is passed into this parameter. Use `std::vector` instead. For example:
```cpp
void foo(const int a)
{
    int arr[a]; // Compilation error, regardless of what arguments `foo` is called with.
    std::vector<int> vec(a); // Ok.
}

int main()
{
    foo(42); // It doesn't matter that `42` is known at compile-time; `int arr[a]` causes a compilation error regardless.
}
```
### `const` return type

In short, never make the return type const.

On some types (such as `int`), it has absolutely no effect. Your compiler might even warn you about this. For example, Clang says this:
```cpp
const int foo()
{
    return 42;
}
```
```
<source>:1:1: warning: 'const' type qualifier on return type has no effect [-Wignored-qualifiers]
    1 | const int foo()
      | ^~~~~
```
On other types, such as `struct`s, `std::vector`/`std::string`, etc, it does two things. Firstly, it prevents assigning to temporaries produced by calling the function. You'll sometimes see it in very old code:
```cpp
int a() {return 42;}
std::string b() {return "foo";}
const std::string c() {return "foo";}

int main()
{
    a() = 43;    // Compilation error, can't assign to temporary.
    b() = "bar"; // Compiles, but does nothing useful.
    c() = "bar"; // Compilation error, can't assign to a `const std::string` temporary.
}
```
Note how assigning to an `int` temporary is a compilation error (which is helpful, since it makes little sense), but assigning to temporaries of struct types and most `std::` types *does* compile. But it doesn't do anything useful, since the temporary is soon destroyed, so the new value is lost.

In the old C++98 days people used `const` return types to prevent this behavior, but we no longer do this.

Starting from C++11 (so since 2011), it also has another effect of silently making some operations on the function result *slower*, which is why it's no longer popular:

```cpp
std::string a() {return "some very long string here";}
const std::string b() {return "some very long string here";}

int main()
{
    std::string s;
    s = a(); // Fast.
    s = b(); // Slow.
}
```

You'll learn more about this effect in later chapters.

You'll also learn how to disable this unwanted behavior (being able to assign to temporaries) for your own types, we have better ways of doing this now. And for `std` types it can't be disabled, so you just have to be careful.
