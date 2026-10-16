# A
# "Anti-pattern"

A [pattern](#pattern) that's considered to be a bad idea.

# B
# C
## "Conforming"

[revise](#TODO)
An [implementation (sense 2)](#implementation) is conforming if it accepts [valid programs](#well-formed) and rejects [invalid programs](#ill-formed); and programs compiled with it behave as the [standard](#standard) says they should behave.

If a compiler doesn't follow the standard in some situation, then its behavior can be called "non-conforming" in that situation.

## "Class"
A type created using `class X {};`, `struct X {};`, or `union X {};`.

People usually don't call `union`s classes, and `std::is_class_v` reports `false` for them, but they still *are* classes ([source](https://eel.is/c++draft/class#nt:class-key)), and can have member functions, constructors, etc.

`struct` also creates a class, and people *do* call them such (this glossary included). The only difference between a `struct` and a `class` is the default access (`public` vs `private` respectively) for members and bases.


## "CRTP"
Stands for [Curiously Recurring Template Pattern](#curiously-recurring-template-pattern).

## "Curiously recurring template pattern"
The practice of inheriting a class from a class template, to be exact), passing the former class as a template parameter to it. For example:
```cpp
template <typename T> struct A {};
struct B : A<B> {}; // <-- here
```

[Read detailed explanation.](curiously_recurring_template_pattern.md)

It's "recurring" because `B` is repeated twice in `B : A<B>`. It's a "pattern" because it's a programming [pattern](#pattern).



# D
# E
# F
# G
# H
# I
## "IF"
Can stand for [ill-formed](#ill-formed).
## "IFNDR"
Stands for [ill-formed, no diagnostic required](#ill-formed-no-diagnostic-required).
## "Ill-formed"
[](#TODO)
A program is ill-formed if it's invalid and shouldn't compile.

## "Ill-formed, no diagnostic required"
[](#TODO)
## "Implementation"
**Sense 1:** The process of making a program (or its part) according to some plan or description, or the resulting program (especially as opposed to the original plan/description).

**Sense 2:** Everything needed to compile C++ programs: a [compiler](#compiler) together with a [standard library](#standard-library), etc, which are made as described in the C++ [standard](#standard). This is just a special case of sense 1 that comes up often.

**Sense 3:** A function [definition](#definition) as opposed to its [declaration](#declaration). For example:
```cpp
// In 1.h:
void foo();
```
```cpp
// In 1.cpp:
void foo() {/*...*/} // <-- This is the implementation of foo.
```

# J
# K
# L
# M
## "Meyers' singleton"

Refers to storing a [singleton](#singleton) instance in a `static` variable in a function:

```cpp
MyClass &instance()
{
    static MyClass ret;
    return ret;
}
```

This is as opposed to global variables, or `static` variables in classes. Unlike those, `static` variables in functions are initialized on the first use, which prevents [static initialization order fiasco](#static-initialization-order-fiasco).

And unlike naive manual attempts to create a class instance on first use:
```cpp
// This is a bad non-thread-safe implementation, don't use!
MyClass &instance()
{
    static MyClass *ret;
    if (!ret)
        ret = new MyClass;
    return *ret;
}
```
...Meyers' singleton is thread-safe, meaning if multiple threads call `instance()`, one of them will construct it and others will wait until it finishes constructing. `static` variables in functions do this automatically.

Named after [Scott Meyers](https://en.wikipedia.org/wiki/Scott_Meyers) who popularized it.

Some people call any use of `static` variables in functions (as opposed to global ones or `static` in classes) a "Meyers' singleton", even if their types are not [singletons](#singleton), but this is inaccurate.

# N
# O
# P
## "Pattern"
In programming, a pattern is a specific trick/approach to solving a specific kind of problems.

They can be specific to one programming language, such as C++'s [CRTP](#crtp), or not depend on the language (then they're often called ["design patterns"](https://en.wikipedia.org/wiki/Software_design_pattern)).

## "Polymorphic class"
A class is polymorphic if it has at least one `virtual` function. `virtual` destructors count as virtual functions. Inheriting a `virtual` function (or desturctor) also counts.


## "Polymorphism"
**Sense 1:** In C++, it usually means the practice of using [polymorphic classes](#polymorphic-class).
[](#TODO)
## "Pure virtual"

A [class](#class) member function is pure virtual if it's declared with `= 0`. Only `virtual` functions can be marked `= 0`.
```cpp
struct A
{
    virtual void foo() = 0;
};
```

Simply speaking, those are intended to be overriden by derived classes.

Having at least one pure virtual function makes a class [abstract](#abstract), i.e. instances of it can't be created. This includes pure virtual functions inherited from a base, unless overridden to be non-pure-virtual (so overridden to be normal functions).

There are no "pure virtual classes". That's an incorrect way to refer to an [abstract](#abstract) class.

Contrary to popular belief, a derived class doesn't *have* to override pure virtual functions. If it doesn't, it just stays abstract.

A destructor can be pure virtual. Then any derived class is considered to override it, even if it doesn't specify a custom destructor (so it doesn't make derived classes abstract).

Contrary to popular belief, pure virtual functions can have bodies. But they must be defined outside of the class body:
```cpp
struct A
{
    virtual void foo() = 0;

    // virtual void bar() = 0 {/*...*/}  // This is a compilation error. Except that notably the MSVC compiler doesn't check this.
};

void A::foo() {/*...*/} // ok
```

Those bodies can only be executed when calling the function using this syntax: `x.A::foo();` (in general adding `A::` disables `virtual`-ness of the call).

Attempting to call a pure virtual function without `Base::` (which can only happen if you try to call it from the constructor or destructor of the abstract base class; there the overrides in the derived classes are not considered) causes [undefined behavior](#undefined-behavior), and typically causes a runtime error.

# Q
# R
# S
## "Singleton"

A programming [pattern](#pattern): a class that can have at most one instance, together with a way to access that instance.

Usually this is a class with a `private` constructor (so the user can't create more instances), and with a `static` function that returns a reference to its only instance:
```cpp
struct A
{
    // ...

    static A &instance()
    {
        static A ret;
        return ret;
    }

    A(const A &) = delete;
    A &operator=(const A &) = delete;

  private:
    A() = default;
};
```

Also see [Meyers' singleton](#meyers-singleton).

Some people consider all global/static variables "singletons", even if they have types that can have more than one instance. This is inaccurate.

## "SIOF"

Stands for [Static Initialization Order Fiasco](#static-initialization-order-fiasco).

## "Specialization"
**Sense 1:** A template with specific template arguments substituted into it.

For example, `std::vector<int>` is a specialization of `std::vector`. Here `std::vector<int>` is a class, and `std::vector` is a class template (which is not a class, only its specializations are).

**Sense 2:** An alternative version of a template that is used intead of the main one for some template arguments. Either an [explicit specialization](#TODO) (which fixes all template arguments)

## "Struct"

In C++ `struct` creates a [class](#class). There are technically no structures in C++, only classes (unlike in C). `std::is_class_v` returns true for structs.

The only difference between a `struct` and a `class` is the default access (`public` vs `private` respectively) for members and bases.

# T
## "Templated"

An entity is templated if it's either a template or declared within a template. For example:
```cpp
template <typename T>
struct A
{
    void foo() {}
};
```
Here, `foo` is templated but is not a template. If would be a template if it had a `tempalte <....>` of its own.

`A` is both templated and a template.

[Source.](https://eel.is/c++draft/temp.pre#:entity,templated)

# U
## "UB"
Stands for [Undefined Behavior](#undefined-behavior).

## "Undefined Behavior"

Breaking certain rules of the language at runtime is said to cause "undefined behavior". This can have any effect, including appearing to work normally, or stopping the program with or without an error message, or doing other weird things.

For example, UB is caused by: indexing an array out of bounds, dereferencing a null pointer, etc.

By definition, **UB is something that the [C++ standard](#standard) describes as UB (or doesn't describe at all)**. If it happens to work fine in some cases, it's wrong to say that it's "not UB in those cases". UB doesn't mean "unsafe", UB is what the standard calls UB.

UB can "time-travel": as soon as it becomes inevitable, the program can begin misbehaving. E.g. a program can refuse to enter an `if` branch that contains undefined behavior.

UB usually belongs to one of the following categories:

* Outright nonsensical actions, such as derefercing null pointers.

* Things that look like they should work, but compilers optimize with the assumption that you don't do this. For example:

  ```cpp
  int x[2][2];
  x[1][1] = 42;
  std::cout << x[0][3] << '\n';
  ```
  This will usually print `42`, because `int[2][2]` has the same memory layout as `int[4]`, and `x[1][1]` corresponds to the third element. But compiler optimizations can and will break this, [see here](https://gcc.godbolt.org/#z:OYLghAFBqd5QCxAYwPYBMCmBRdBLAF1QCcAaPECAMzwBtMA7AQwFtMQByARg9KtQYEAysib0QXACx8BBAKoBnTAAUAHpwAMvAFYTStJg1DIApACYAQuYukl9ZATwDKjdAGFUtAK4sGEs6SuADJ4DJgAcj4ARpjEIADspAAOqAqETgwe3r7%2ByanpAiFhkSwxcYl2mA4ZQgRMxARZPn5cAZXVArX1BEUR0bEJtnUNTTmtQ929JWUJAJS2qF7EyOwc5gDMocjeWCbrbgoE%2BKgAdAh72CYaAIKhBADU/KgQd/eqsybxVjf3v/evAE8TABWCxmEEAERBYMheyhX0%2BUPW32uf3uQNBGlhoNUsPWEMk4ORV1Rf2ImAISwY6OhXGxFixwKRKMRJNeLCYoQgHwRPz%2BSWIdyoEHM4LMwLwILcDFFMmeZlmH2JN0RHHmtE4wN4fg4WlIqE4bms1nuCkWy0w9w2PFIBE0avmCEwTCwcW5pAA1iBJPETgAOYEATi4Gj9GkDZg0Zi4gYAbPpOJJeCwJBoNKQdXqDRxeAoQOm7bq1aQ4LAYFByxAkGgWEk6LFyJQa3X6HFgFxgVw%2BHQCLE8xAovbSFFQvUAZwbSPmMQAQB5KLaKqFm01tiCWcMWjjoukLBRLzANxiWh57i8LAcoziHf4cnVABumFPeswqiqXl7Q7umA1O9oeCiYgxw8LAhwIQUUzPUhH2IKJUkwCFMEvYB/yMe15ioAxgAUAA1PBMAAd1nJJGAnXh%2BEEEQxHYKQZEERQVHUHddC7Aw0NMSxrH0AC80geZUCSRwBFPXhUBgwUsF4912iEvwIFcUYWkCBh0Cmfo4i7FI0lkxS9C0goGDU0oBi7GSamGRpPGaPQzM6CyjJmUyLN0pzJlCPpjI0%2BYzSWFYJHVTVtSHbM3j9WMAFpY0ke5gGQZB7g7E4uHuCBcEIEgrTMdYuFmXhCy0RVSCdF0BndL1gXTX8k1IFMKozYLOFzfNbXQktKyQRYCCSD9GwgZt62IcJWFWVQwsi6LtkMYB7mjJKThffAiAkvQKOEURxFo1aGLUIcWNIAigKSMiAo4LV6p3bNZw/bqHlQKhQoiqKYrihLgSSlKPFrAbMuy3KWqLQritdSgTuq2r00zUTGtsZr8odT0QDq391iCi7obh%2BZSwreAqxQVAvtbXr%2BtbEB207btaF7Yh%2B0HHcpzHMjh1HGd50XBxGdXRgCA3Lchz3A8j1oE9GYvKbrz1W8lzwR9n14V930/Hdv1/PV/0A4CMFWPVwLwSCbRguClEQ5DUNAAG%2BCw3D8KIkidRtVaqI26QtqUHbmJAAI2OMY1LG4qIpP4wSMlPcLZxR/VxLwST4G8n8peceSVJc5TVPc6YTLybSMmT/TZIcjPbIYLoRissZbDjjoi/stP1Js5zS6Uw43OKWucoWXyaJOs7If1TgHvG574sS5LUsWjLrT%2BjHHWdYGysRyrE2TefzqzaG8wLVqsdxzqbqJ/GW1iIa2E4UbHui2Kh7em1MDH5auwd9aaOd2RtqYvVdACA6mCOs8u9R1eOBXS6h%2Be4d1%2B5PQvq9d6EBPoH2ID9BUeV0LTxKm6UGS86o92zE1DeAN5jlQXhwcOWD0bIIRsCMwSVJChmDIGDQ8RAzxDDOsBMHAzBLxDBDBqOZ/oFROuwleUMeFT2gn2DI3ogA).

* Things that only work on the certain OSes, kinds of processors, etc, that could work for you but break for someone else.

* Things that are in fact completely safe, but are made UB out of caution.

  For example, taking a pointer to a class member, and subtracting [`offsetof`](https://ru.cppreference.com/cpp/types/offsetof) from it to get a pointer to the entire class. People actively rely on this behavior, compilers will never break it, yet the standard is being cautious.

  Those rules tend to be relaxed over time in the standard to match existing practice. For example, this happened to [implicit lifetimes](#implicit-lifetimes). This was illegal, yet people were doing it anyway and compilers didn't mind, so it was made legal.

Some things are said to cause UB at compile-time, but this is widely considered to be a wording defect in the standard. They should be made [IFNDR](#ifder) instead, this is gradually being fixed. "Compile-time UB" is just a stupid way to say IFNDR.

# V
# W
# X
# Y
# Z
