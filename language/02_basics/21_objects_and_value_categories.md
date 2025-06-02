# Objects and value categories

This is a theory-heavy topic, so I've been putting it off. But it is very important and needs to be explained.

What do the following statements have in common?

* `… = 10;`

* `std::cin >> …;`

* `foo(…);` (where `foo` is `void foo(int &ref) {/*...*/}`)

They have a particular requirement for what `…` can be.

`42 = 10;` doesn't compile, but `int x; x = 10;` does. Similarly, `std::cin >> 42;` doesn't compile, but `int x; std::cin >> x;` does. Similarly for `foo(…)`.

So does `…` need to be a variable? Close, but not entirely correct:

```cpp
int arr[10];
arr[0] = 10;
std::cin >> arr[0];
foo(arr[0]);

struct A {int x;};
A a;
a.x = 10;
std::cin >> a.x;
foo(a.x);
```

This compiles, but `arr[0]` is not a variable. Only the entire array is a variable, because that's what we explicitly *declare*. Its individual elements can't be called variables. The individual struct members like `a.x` also can't really be called "variables", as only the entire `a` is a variable.

So what is the actual requirement?

I need to introduce some new concepts before I can explain this properly.

## Objects

When a program manipulates values at runtime, those values are held in something called **objects**, which exist at runtime too. *Variables* are just words/names in the source code that represent those objects existing at runtime.

For example:
```cpp
int x = 10;
x = 20;
```
When this program runs, an object gets created for the variable `x`, with the value `10`. The value of that object then gets changed to `20`, but it's still the same object. Lastly when the execution leaves the scope of `x`, its object gets destroyed.

Strictly speaking, a variable itself is just something spelled (declared) in the source code. It's not something that exists when the program runs, only the objects exist. A variable is just a word in the source code. (Here readers with prior programming experience are probably raising their eyebrows, but this is true, [here's proof](https://stackoverflow.com/a/79566664/2752075).)

But *of course*, we use the words loosely. Those are very high levels of pedantry. We often say "variable" to mean "the object associated with the variable". We don't say "the object associated with variable `x` has value `42`", we say just "variable `x` has value `42`", even if at a very pedantic level it's incorrect.

And you don't have to be pedantic in your speech in this case, even this tutorial itself isn't in most cases. But please try to understand the difference between those concepts.

In the source code, we use **expressions** to refer to objects. Expressions are parts of the source code that represent objects existing at runtime. After you declare a variable, its name can act as an expression.

Array elements and struct members are objects too, and `arr[i]` and `boss.health` are expressions referring to them. And `arr` and `boss` are expressions as well, referring to the entire array/struct.

And lastly, there are *some* expressions don't refer to any objects, for example the calls to `void` functions, which are expressions of type `void`.

## Temporary objects

But literals like `42` are expressions too, they too represent objects. Same for expressions like `10 + 20` or `double(42)`, they also represent objects. (Again, I'm omitting some nuances here, that's for later.)

Those are called **temporary objects**, or **temporaries**. Unlike (the objects associated with) variables which are created manually (via a declaration), temporaries are created automatically as needed to hold the results of computations (or values of numeric literals, etc), and then die off quickly. **Temporaries are normally destroyed at the end of the *full-expression* they were created in.** (Revisit the ["computations"](./03_computations.md) chapter if you forgot what a full-expression is. This is roughly equivalent to them being destroyed at the end of the line of code they were created in.)

## Lvalues and rvalues

You'll also hear the words **lvalue** and **rvalue** a lot. Roughly, **an lvalue is** (usually) **an expression that refers to a non-temporary object**, and **an rvalue is** (usually) **an expression that refers to a temporary object**.

"Temporary-ness" vs the lack of it applies to **objects** (which is what exists at runtime), whereas "lvalue-ness" vs "rvalue-ness" applies to **expressions** (the pieces of source code).

"lvalue-ness" vs "rvalue-ness" is called the **value category** of an expression.

### Nuances

Of course there are nuances to this:

* First of all, in some rare cases the value category will not match temporary-ness (you'll have rvalues referring to non-temporaries and lvalues referring to temporaries). This is rare though, for now you don't have to think about it.

* Rvalues are further split into two different categories, so the total number of categories is 3. This sepration rarely matters though, and can't be explained this early in the tutorial.

## Summary

So now we can finally go the example in the beginning of this chapter:

> What do the following statements have in common?
>
> * `… = 10;`
>
> * `std::cin >> …;`
>
> * `foo(…);` (where `foo` is `void foo(int &ref) {/*...*/}`)

What they have in common is that **`…` must be an lvalue**.

And also it must also be non-const, so for example:
```cpp
const int x = 0;
x = 10; // Compilation error!
std::cin >> x; // Compilation error!
foo(x); // Again, compilation error!
```
Those don't work, because `x` is `const`, despite being an lvalue.

### More nuances

Of course, there's more.

Some types allow lvalues to be assigned to, e.g. most (all?) `std::...` types:
```cpp
std::string foo() {return "hello";}

int main()
{
    foo() = "blah";
}
```
This compiles, but doesn't do anything useful, because the modified string, being a temporary, dies at the end of the full-expression.

There's no technical reason for this, just something that the authors of `std::string` forgot (or didn't bother) to disable.

But note that `>>` and passing to reference parameters still doesn't compile even for rvalue `std::string`s. Only assignment does.
