## Curiously recurring template pattern

CRTP happens to allow behavior similar to that of virtual functions. Compare:

### (1)
```cpp
struct A
{
    virtual std::string MyName() const = 0;
    void SayName()
    {
        std::cout << "I am " << MyName() << '\n';
    }
};

struct B : A
{
    std::string MyName() const override
    {
        return "B";
    }
};

int main()
{
    B b;
    b.SayName(); // `I am B`
}
```
### (2)
```cpp
template <typename Derived>
struct A
{
    void SayName()
    {
        std::cout << "I am " << static_cast<Derived &>(*this).MyName() << '\n';
    }
};

struct B : A<B>
{
    std::string MyName() const
    {
        return "B";
    }
};

int main()
{
    B b;
    b.SayName(); // `I am B`
}
```

CRTP removes the overhead of virtual functions, which is good.

But with CRTP you can no longer do [polymorphism (sense 1)](README.md#polymorphism), i.e. use pointers/references to base classes to refer to instances of different derived classes. For example, the following works with [(1)](#1) but not with [(2)](#2):
### (3)
```cpp
std::vector<A *> v; // (2) would cause a compilation error here, because `A` needs template arguments.
v.push_back(new B);
v.push_back(new C); // Some other class derived from `A`.
for (A *a : v)
    a->SayName();
```

It's possible to combine CRTP with polymorphism: inherit the template base from another non-template base, and add some [pure virtual](README.md#pure-virtual) functions to that new base, and implement them in the template:
### (4)
```cpp
#include <iostream>

struct A
{
    virtual ~A() = default;
    virtual void SayName() = 0;
};

template <typename Derived>
struct AA : A
{
    void SayName() override
    {
        std::cout << "I am " << static_cast<Derived &>(*this).MyName() << '\n';
    }
};

struct B : AA<B>
{
    std::string MyName() const
    {
        return "B";
    }
};

int main()
{
    A *b = new B;
    b->SayName(); // `I am B`
}
```

Like [(1)](#1), this makes [(3)](#3) work. But this is also faster than [(2)](#2), because when `SayName()` calls `MyName()`, that call is no longer virtual.

---

Removing overhead is not the only reason to use CRTP. CRTP can do some things that `virtual` functions can't. For example, when the base class methods need to have the derived class as a return type or a parameter type:

### (5)
```cpp
template <typename Derived>
struct Addable
{
    friend Derived operator+(Derived a, const Derived &b)
    {
        a += b;
        return a;
    }
};

struct A : Addable<A>
{
    int x = 0;
    A(int x) : x(x) {}

    A &operator+=(const A &other)
    {
        x += other.x;
        return *this;
    }
};

int main()
{
    A result = A(10) + A(20);
    std::cout << result.x << '\n'; // 30
}
```

Here, `Addable` can be used to add `operator+` to a class that already implements `operator+=` (e.g. `A`), reducing the amount of boilerplate needed in `A`.

---

In CRTP, the template base can have additional template arguments, they are sometimes useful.

In CRTP, the derived class can be a template too, then it must be passed to `A` with its template arguments:
```cpp
template <typename U, typename V> struct B : A<B<U,V>> {};
```
