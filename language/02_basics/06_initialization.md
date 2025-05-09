# Initialization

In the previous chapters, when we declared a variable, we've always set the default value for it.

Which doesn't always make sense, because e.g. in:
```cpp
int x = 0;
std::cin >> x;
```
...it immediately gets overwritten, so there's little point in setting the initial value.

The good news is that it's optional. You can just do:
```cpp
int x;
std::cin >> x;
```

Setting the value when declaring a variable is called **initialization**, as in `int x = 42;`. And the value itself (`42` here) is called an **initializer**.

And modifying a variable *after* it was declared is called **assignment** (not initialization).

```cpp
int x = 10; // Initialization.
x = 20; // Assignment.
```

## Uninitialized variables

What is the value of a variable if you didn't initialize it?

Some languages zero all variables by default, but C++ doesn't.

In C++, if you try to read an uninitialized variable, one of several bad things can happen. You could get some arbitrary value, that's what happens most often.

You could get a runtime error (an error when the program is running, as opposed to during compilation) if you enabled the optional checks that check for this (if you chose to follow my [tooling tutorial](https://github.com/holyBlackCat/cpp-tutorials) and are using the Clang compiler, you can enable those checks and observe the errors by enabling the [`-fsanitize=memory` compiler setting](https://github.com/HolyBlackCat/cpp-tutorials/blob/master/tooling/articles/recommended_compiler_flags.md)).

This can have some other adverse effects too, which will be explained in more detail in later chapters. For now just know that you shouldn't do this.

It's not a bad idea to always initialize your variables, to avoid subtle bugs. If you don't have a meaningful value to set them to, zero them.

## Nuances of the word "initialization"

Note that the word "initialization" is often used loosely. In general in programming it has a meaning of "setting the initial (meaning "first") value", while in C++ for variables it has the narrow meaning of setting the value in the declaration. So there are two distinct meanings.

So you'll often see things like:
```cpp
int x;
// Some code that doesn't involve `x`.
x = 42;
// Some code using `x`.
```
...being referred to as "initializing" `x`, in the sense that you give it some value before using it. This isn't entirely wrong, this just uses a different meaning of "initialization".

Another situation where this comes up is that we no longer call `x` uninitialized after assigning to it, despite no actual initialization taking place. This is another example of using the word "initialization" in the loose sense of "setting an initial value".

### Default-initialization

Another annoying nuance is that not initiailizing a variable is technically called "default-initializing" it. This is just a wording quirk, and a funny way to nitpick people.

## Exercise

Create a few uninitialized variables and print them. Hopefully you'll see a few random values.

Now initialize them and print them again.
