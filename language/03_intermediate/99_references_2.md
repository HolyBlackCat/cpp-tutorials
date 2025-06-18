
## More about references

Okay, so now what happens if we do `Increase(5, 100);` instead of passing a variable to the first parameter?

Same thing as what happens if you try this with a non-parameter reference variable:
```cpp
int main()
{
    int &target = 5;
}
```
That being a compilation error.

The reason for this should be intuitively obvious, at least for functions. If you have `void Increase(int &target, int value)`, this implies that you want to access value of the first argument in the caller after calling the function (or you wouldn't have made it a reference), so the compiler prevents you from using the function "incorrectly".

In other words, if `Increase(5, 100)` compiled and updated the `5` to `105`, how would you read its new value if it's unnamed (isn't a named variable)?

But...

## Value categories

...it is interesting to explore what happens here on a deeper level.

Why *exactly* is `x` (given `int x;`) a valid initializer for a reference variable (or a valid argument for a reference parameter), but `5` isn't?

You'll also notice that
```cpp
int x;
std::cin >> x;
```
...is valid, while `std::cin >> 5;` isn't.

Similarly, `x = 10;` is valid but `5 = 10;` isn't.

Clearly there's a more general rule at play here.

You might assume that those things are only legal for variables, but that's clearly not it, because array elements are not variables (only the entire arrays are), but can be `std::cin >>`ed into, assigned, etc. And the same is true for the struct members (`boss.health`), and those aren't really variables either (only the entire struct variable is a variable).

### lvalues and rvalues

Variables (like `x`) are **lvalues**. If your variable is an array or a struct, then its elements/members are also lvalues.

Numeric literals (like `5`) are **rvalues**. Results of most operators are rvalues, such as `10 + 20`. Results of most casts are rvalues too, such as `double(42)`.

"lvalue" and "rvalue" are **value categories**.

This is the missing component. `5 = 10;` doesn't compile because `5` is an rvalue, and `std::cin >> 5;` doesn't compile for the same reason. `int x;` `x = 5;` compiles because `x` is an lvalue, and so on.

"lvalue" originally stood for "left value", meaning it can be the lhs (left operand) of `=` assignment. But it can be the rhs too, of course.

"rvalue" stood for "right value", meaning it can only appear on the right side of an assignment.

Not all lvalues are assignable, so "can be the lhs of assignment" isn't a proper definition of lvalue. For example, `const` variables are still lvalues, but can't be assigned. Arrays can't be assigned either, despite being lvalues. Same for structs with const members, and those const members themselves.

A const variable can't be passed to an `int &target` parameter, or used with `std::cin >>`, so being an lvalue isn't the only requirement for those. Additionally they require the lack of constness.

### Summary

Lvalues (lvalue expressions) usually refer to things that will live for some more time. Whereas rvalues (rvalue expressions) usually refer to things that are about to be destroyed. Variables live for some time. Literals such as `5` don't exist outside of the expression they're used in.

There's more to this classification than just "lvalues" and "rvalues". Rvalues are further split into two categories, but they aren't very important, especially now, and can't be explained properly this early in the tutorial.

> ## Exercise 4
>
> Take one of the programs you've written before, and find every lvalue in it, then every rvalue.
>
> Don't worry *too* much about memorizing this yet. A better understanding will come naturally as you use the language more, and as you learn the more advanced features that rely on those concepts.










## Const references

This appears contradictory at face value, but we do have const references in the language.

```cpp
void foo(const int &x)
{
    std::cout << x << '\n';
}
```
You can't assign to `x` in the function, so what's the point?

The point is that copying non-reference parameters can be expensive. Let's say you have a `std::vector<int>` with a few million elements, and want to pass it into a function. If you make a parameter of type `std::vector<int>`, calling the function will copy the entire vector, which is unnecessarily slow (unnecessarly, unless you actually need a copy for some reason), not to mention doubling the memory usage.

Using a reference `std::vector<int> &x` would get rid of the copying, but isn't a good solution either, because this function can no longer accept constant vectors (for no good reason, if it's not going to modify them), and also can't accept rvalue vectors (`foo({1, 2, 3});`, which is convenient and legal if the parameter is a non-reference vector; note that the lone `{1, 2, 3}` isn't a vector by itself, but can serve as an initializer for one), for the same reason as why you can't pass a literal into an `int &x` parameter (see above).

Const references solve all those problems. Passing arguments to them doesn't involve copying. Literals can be passed to them (and rvalues in general, see `foo({1, 2, 3})` above). Const variables can be passed too.

**Non-const references can only be initialized with non-const lvalues. Const references can be initialized with anything.**

## Const non-reference parameters

Okay, now what about `int foo(const int x)`?

When a parameter is `const` and isn't a reference, this is something that doesn't affect the caller at all.

This only matters inside the function. Do this if the function is particularly long and you're worried that you might accidentally modify a parameter that you're not supposed to. The caller won't see the change either way, but if *you* are reading the parameter multiple times in the function, then it might matter for you inside the function.

This concludes the part about passing parameters.

> ## Exercise 5
>
> Experiment with const reference and const non-reference parameters.













#### Dangling `const` references

`const` references are easier to dangle.

While `int &foo() {return 42;}` doesn't compile, because `42` is an rvalue, the same thing with a `const` reference does* compile:

```cpp
const int &foo()
{
    return 42;
}
```

This immediately dangles, so reading the result is always UB.

\* Newer C++ versions (C++26) make the above into a compilation error, since it's an obvious mistake.

> ## Exercise 8
>
> Experiment with UB caused by dangling references, and try to observe the incorrect values produced by it.
>
> Enable the [address sanitizer](/tooling/articles/recommended_compiler_flags.md) and see if it catches this error.
