# References (part 2)

As you might have guessed by now, non-const references can only bind to (be initialized with) non-const lvalues:

```cpp
int x;
const int y = 42;
int &a = x; // ok
int &b = 43; // error, because 42 is an rvalue
int &c = y; // error, because 42 is const
```

The rationale for rejecting const was already explained in ["references (part 1)"](./16_references_part_1.md). Allowing that would allow you to inadvertently modify const objects through non-const references.

But the rationale for rejecting rvalues is more subtle.

Astute readers might assume that it is because the reference would supposedly dangle, as the temporary it's pointing to would get destroyed immediately, and the reference would outlive it (as a reminder, reading/writing to dangling references is UB).

A good assumption, but then why does `const int &x = 42;` compile? Wouldn't it dangle too?

So there is more to it. This has to do with **functions**:

## Reference parameters

Consider how references can be used as parameters:
```cpp
void a(int x) {}
void b(int &x) {}
void c(const int &x) {}
```

```cpp
int x = 42;
a(x); // ok
b(x); // ok
c(x); // ok
a(42); // ok
b(42); // error, a reference can't bind to an rvalue
c(42); // ok
```

Do those examples make sense?

Remember *why* we are using references:

* Non-const references allow the function to modify the parameter, and then allow the caller to **see the updated value**.

* Const references are used primarily to avoid expensive copies (`int` is dirt cheap to copy, but consider a `std::vector<int>` with a few million elements), when the function doesn't *need* to modify the parameter.

Disallowing `b(42);` helps prevent misuse of the function, because how would the caller get the new value of the argument, if they passed a temporary that died immediately after the call?

In other words, since the function `b` has a non-const reference parameter, it's probably intended to be used like this:
```cpp
int x = 42;
b(x); // Might modify `x`!
std::cout << x << '\n'; // Use the new value somehow, this might no longer be 42.
```
So `b(42);` makes no sense, as you're missing out on the ability to see the updated value, so the compiler prevents this misuse.

As for const reference parameters, `c(42);` is perfectly fine, because the function can't modify the argument.

And there are no dangling issues in `c(42);`, because the temporary (`42`) outlives the call *anyway* (it lives until the end of the full expression, which roughly means until the end of the line of code).

## References outside of parameters

So it should be clear now why `b(42)` errors but `c(42)` is allowed. But what about this?

```cpp
int &x = 42; // error, because 42 is an lvalue
const int &y = 43; // ok
```

First of all, it has to work like this for consistency with function parameters. The rules for parameter initialization as the same as those for variable initialization.

This conveniently happens to prevent dangling for `x` too. But what about `y`?

### Lifetime extension

`y` doesn't dangle because of a little-known rule of **lifetime extension**. If a reference binds to a temporary, it extends its lifetime to match that of the reference.

So `const int &y = 42;` if a funny way of spelling
```cpp
const int some_hidden_name = 42;
const int &y = some_hidden_name;
```
Which is in turn a funny way of spelling `const int y = 42;`.

**This isn't a feature you should use intentionally.** It comes with a [long list of rules](https://en.cppreference.com/w/cpp/language/reference_initialization.html#Lifetime_of_a_temporary) about when it does or doesn't work, and those rules disallow all the interesting usecases. So in the end, this doesn't give you any new possibilities compared to just using the normal non-reference variables.

For example, this still dangles:
```cpp
const int &foo() {return 42;}
```
And in fact this specific example now causes a compilation error in C++26, instead of silently dangling.

This rule (the lifetime extension) is only there to cover your back and prevent silly mistakes, such as:

```cpp
const std::string name = "Mr. Doom Guy";
const std::string &GetName() {return name;}

int main()
{
    const std::string &player_name = GetName();
}
```
So far so good, no lifetime extension here. Now imagine that `GetName` gets changed to:
```cpp
const std::string name = "Doom Guy";
std::string GetName() {return "Mr. " + name;}
```

Now `const std::string &player_name = GetName();` continues to work, which is convenient.

It is nice to not have to go update all callers after making this change to the return type.
