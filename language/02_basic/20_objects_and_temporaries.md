# Objects and temporaries

## Variables and values

I need to introduce some new terminology.

You already know that variables hold values, e.g. given `int x = 42;`, `x` holds 42.

But you also encountered values not stored in variables. For example, in `cout << 10 + 20;`, neither 10 nor 20 nor the resulting 30 are stored in variables. (Because there are no [declarations](04_variables_1.md#variable-declarations) here.)

Or if you have an array:
```cpp
int a[3] = {1, 2, 3};
```
The individual elements `a[i]` (`a[0]`, etc) are not variables either. Only the entire `a` is a variable.

Similarly with a struct:
```cpp
struct A {int x, y;};
A a = {42, 43};
```
`a.x` and `a.y` are not variables, only the whole `a` is a variable. (`x` and `y` are also not variables, as they don't really exist by themselves, outside of specific instances of the struct.)

## Objects

All those things storing values are called "objects".

An **object** is something that exists at runtime and has a value. A C++ program creates, destroys, and changes values of objects while it is running.

Notice that when assigning to an object (to a variable, array element, etc), it remains the same object, but its value changes.

There's a lot of nuance to this, and the exact meaning of "objects" and "values" can't be explained this early in the tutorial. For now this explanation should be enough, I will revisit this later.

## Variables vs objects

Most variables are objects (but not all objects are variables). E.g. given `int a, b[3];`, both `a` and `b` are both variables and objects (and `b[i]` are objects but not variables).

The only variables that are **not** objects are references: they merely refer to objects that exist elsewhere.

You could say that a variable is a **named** object (or a reference), but this is not entirely accurate (e.g. function parameters can be unnamed, which makes them impossible to access, but they are still variables).

A better definition is that a variable is a **declared** object (or a reference). (E.g. in `int a[3];`, you *declared* `a` to be an array, which makes it a variable.) This is close enough, but not fully accurate still.

Technically, a variable is just a label in the source code (at compile time) for an object that exists at runtime. So if you have `int x = 42;`, it's technically wrong to say that "the value of `x` is 42", because it's the value of the object corresponding to `x` (referred to by `x`), not of the variable `x` itself, which is just something spelled in the source code, not something that exists at runtime. ([Source.](https://stackoverflow.com/q/79566041/2752075))

Most programmers are not aware of this distinction, and it usually doesn't matter. Most people use the word "variable" loosely, conflating it with the object corresponding to the variable. This tutorial does this too; there is no point in always being pedantic about it, as long as you do it intentionally and understand the correct meaning of words.


## Temporary objects

There are certain objects that are destroyed almost immediately after being created, rather than [at the end of scope](./09_if_else.md#variables-in-the-if-bodies). They are called **temporary objects** or "**temporaries**". Those are:

* Numeric literals, such as `42`, `1.23`, `true`/`false`, `'A'`. (Notably not [string literals](./17_strings.md#string-literals) such as `"abc"`; those exist for the entire duration of the program, for reasons that will become apparent later.)

* The result of calling a function, if the function returns by value (so not a reference and not `void`).

* The result of a cast, such as `double(42)` (assuming you're not casting to a reference type, the purpose of such casts will be explained later).

Variables are never temporaries.

(I'm simplifying things a bit for now, advanced readers should set aside their pitchforks.)

Temporaries are normally destroyed at the end of the [full expression](./03_expressions.md#expressions-are-composable) that created them (rather than at the end of scope). This usually means that they die at the end of the current line, at `;`.


## Why do we care about temporaries?

Primarily because if you manage to form a reference to a temporary, that reference will almost immediately dangle.

The compiler tries to hold your hand and prevents the obvious mistakes:
```cpp
int &blah = 42; // Compilation error, can't bind a reference to a temporary.
```
But it doesn't catch everything:
```cpp
#include <iostream>
#include <vector>

std::vector<int> foo()
{
    return {1,2,3};
}

int main()
{
    int &blah = foo()[0];
    // `blah` is dangling here.
    std::cout << blah << '\n'; // UB!
    blah = 42; // UB!
}
```


## More ways of creating temporaries

There are some convenient ways of creating temporaries that weren't mentioned before.

For example, with vectors:

```cpp
std::vector<int> v = {1,2,3};

if (v == std::vector<int>{1,2,3})
    std::cout << "Equal!\n";
```
```cpp
void foo(std::vector<int> v) {/*...*/}

int main()
{
    foo(std::vector<int>{1,2,3});
    foo({1,2,3}); // This works too!
    foo(std::vector<int>(5)); // Same as `{0,0,0,0,0}`, just like `std::vector<int> vec(5)` fills `vec` with 5 zeroes.
}
```

All the same tricks work with structs:
```cpp
struct Monster {std::string name; int health = 0;};

void foo(Monster m) {/*...*/}

int main()
{
    foo(Monster{"Dragon", 100});
    foo(Monster{.name = "Dragon", .health = 100});
    foo({"Dragon", 100});
    foo({.name = "Dragon", .health = 100});
}
```

In general, `Type{...}` is the unnamed version of `Type variable{...};` (which works similar to `Type variable = {...};`), and `Type(...)` is the unnamed version of `Type variable(...);`. Though there are exceptions to this rule.

So e.g. since `int(42)` and `int(12.3)` are legal, `int x(42);` and `int y(12.3);` are also legal. (And of course `int(42)` is same as `42`, since the type of `42` is already `int`. Unlike `12.3` which is `double`.)

Also notice that braced lists like `{1,2,3}` (or `{"Dragon", 100}` or `{.name = "Dragon", .health = 100}`) have no type of their own. While `Monster{"Dragon", 100}` is a temporary of type `Monster`, `{"Dragon", 100}` has no type (and isn't even considered an expression, since all expressions have types). The compiler guesses the type from the context.

Do you understand why? If you had `struct A {int x, y;};`, then should `{1, 2}` be `std::vector<int>{1, 2}` or `A{1, 2}`? The compiler can only tell from the context.

## Const vs non-const references

Const references add complexity to this.

If you [remember from earlier](./16_references.md#more-corner-cases), while `int &blah = 42;` is a compilation error, `const int &blah = 42;` is allowed. Why? Doesn't it dangle?

This is related to functions and their parameters.

As explained in [the chapter about functions](./19_functions_1.md), references can be function parameters. Const and non-const parameters, despite looking similar, are used for different purposes:

1. Non-const reference parameters allow the caller to [see the updated value](./19_functions_1.md#modifying-the-parameters) after the function has modified the parameter.
2. Const reference parameters (which the function can't change) are instead used to [avoid expensive copies](./19_functions_1.md#const-references-as-parameters).

So let's say we have:
```cpp
void foo(int &ref)
{
    ref += 1;
}

int main()
{
    int x = 10;
    foo(x);
    std::cout << x << '\n'; // 11
}
```
But if you try to do `foo(11);`, that's a compilation error. It makes sense, since because the argument (`11`) is unnamed, even if it compiled, the caller would have no way to refer to the updated value:
```cpp
foo(10); // Imagine this compiled...
std::cout << ??? // Then how do we refer to 11 here?
```
So this rule helps prevent accidental misuse of functions.

And all this is very convenient, because **function parameters are variables**, so passing arguments is governed by the **same rules** as variable initialization. E.g. those two lines error because of the exact same rule:
```cpp
int &ref = 10; // Compilation error.
foo(10);       // Compilation error.
```
So this rule happens to kill two birds with one stone: it both protects against dangling references (`int &ref = 10;`), and *also* protects against accidentally misusing functions that accept reference parameters (`foo(10);`).

Notice that if `foo(10);` compiled, it wouldn't dangle, because the temporary `10` outlives the function call. (It dies at the end of the full expression, and `foo(10)` is the full expression here, so it dies after the call.) This is explained in more details below.

### Const references

Now what about const references? As explained in the [chapter about functions](./19_functions_1.md#const-references-as-parameters), const references serve a different purpose as function parameters, compared to non-const ones. Let's say you have this function that prints a vector: (I'm reusing the example from that chapter)
```cpp
void PrintVector(const std::vector<int> &vec)
{
    for (int x : vec)
        std::cout << x << '\n';
}
```
Its parameter should be a const reference. If you make it a non-reference, printing large vectors will be slow, since the vector would have to be copied into the parameter. Making it a reference avoids the slow copying. And making it a `const` reference (as opposed to non-const) allows printing `const` vectors, not only non-const ones.

But what if you passed a temporary?
```cpp
PrintVector(std::vector<int>{1,2,3});
PrintVector({1,2,3});
```

It's convenient to allow those, and they are indeed allowed. (Compared to non-const reference parameters, there is no updated value that the caller might be interested in, so there is no harm in allowing passing unnamed objects.)

But aren't those references dangling? No, since the temporaries die at the end of the full expression (in `PrintVector({1,2,3});`, the full expression is the entire `PrintVector({1,2,3})`), so they die after the function call finishes.

This isn't entirely fool-proof. Firstly, what if the function stores the reference somewhere? That will dangle. For example:
```cpp
struct A
{
    const int &ref;
};

std::vector<A> vec;

void foo(const int &ref)
{
    std::cout << ref << '\n'; // This is fine, not dangling yet.
    vec.push_back({ref});
}

int main()
{
    foo(42); // Ok.
    // 42 dies here.

    int x = 10;
    foo(x); // Ok.

    std::cout << vec[0].ref << '\n'; // UB, the temporary `42` is already dead!
    std::cout << vec[1].ref << '\n'; // This is fine, `x` is still alive.
}
```
You just have to be careful about this. Normally, if a function takes a const reference, the expectation is that it doesn't try to preserve it after the call. If it does, it should be clearly documented in a comment (and later you'll learn how to make your functions resistant to this).

Secondly, since the rules of passing arguments and for variable initialization are the same, if we allow `foo(42)` (where `foo` is `void foo(const int &ref)`), we must also allow `const int &ref = 42;`.

Doesn't that dangle?

It normally would, but since this is so error-prone, there is a special rule that prevents it. Enter...

## Temporary lifetime extension

In short, the obvious cases like `const int &ref = 42;` are magically allowed. The lifetime of the temporary `42` is extended to match that of `ref`, so it basically becomes `const int ref = 42;` (not exactly, but the minor differences don't matter now).

If you do `const int &ref = foo();` and `foo` returns by value, the lifetime of returned value is also extended.

But this only works in very simple cases. The rule of thumb is that if you have a cool idea how to use the lifetime extension, then it probably doesn't work in that case. It almost never gives you any new possibilities.

E.g. references returned from functions don't extend lifetime:
```cpp
const int &a()
{
    int x = 42;
    return x; // This already doesn't even compile in newer C++, but imagine it does.
}

int main()
{
    const int &r1 = a();
    std::cout << r1 << '\n'; // UB, this is dangling.

    std::cout << a() << '\n'; // This is UB too. `x` dies when `a` returns, not at the end of the full expression that called `a()`.
}
```
But:
```cpp
const int &b(const int &ref)
{
    return ref;
}

int main()
{
    const int &r1 = b(42);
    std::cout << r1 << '\n'; // UB, this is dangling.

    std::cout << b(42) << '\n'; // This is fine, since `42` lives until the end of the full expression, which is the entire line.
}
```
Function parameters in general don't extend anything, because the temporaries passed to them already outlive the function call.

This doesn't work either:
```cpp
struct A
{
    const int &ref;
};

int main()
{
    std::vector<A> vec;
    vec.push_back({42});
    std::cout << vec[0].ref << '\n'; // UB, this is dangling.
}
```

As you can hopefully see, temporary lifetime extension doesn't have much use, and is best avoided (since it's relatively little-known, and the rules for when it does or doesn't happen are complicated). Cases where it's useful are rare.

You might ask, why is it even in the language? Couldn't we just ban `const int &x = 42;` as a special case and not deal with this?

Probably. The slight advantage of lifetime extension is that it allows changing return types of functions from `const Type &` to `Type` without updating all places where the function is called. But I still believe that if C++ was designed today, we wouldn't have lifetime extension, and `const int &x = 42;` just wouldn't compile.

> ## Exercise
>
> Try to catch dangling references with [ASAN and UBSAN](./12_undefined_behavior.md#catching-ub). Observe how the lifetime extension prevents dangling.
