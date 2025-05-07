# Undefined Behavior

There are certain things that you aren't allowed to do at runtime.

For example, you're not allowed to access arrays out of bounds (that is, using indices smaller than `0` or greater or equal to the array size).

In most other languages doing this would either cause a runtime error, or automatically grow the array to that size for you.

Not in C++ though. C++ has a concept of **Undefined Behavior** (often shortened to just **UB**).

When your program performs certain illegal actions at runtime (those said to "cause undefined behavior", out-of-bounds access being one of them), it will not necessarily show an error, like it would in more civilized languages.

Instead, it can do almost anything:

* Continue working as if nothing happened.

* **Crash** immediately (meaning abruptly close the program with or without an error message).

* Crash at some later point, when performing an unrelated action.

* Do something else.

* Any of those effects can "time-travel" to the past, and manifest themselves not when you trigger UB, but merely when it becomes inevitable (such as the program possibly refusing to enter an `if` branch that would always cause UB).

## Demonstating UB

Let's try triggering UB:
```cpp
int arr[100];
arr[101] = 42;
std::cout << arr[101] << "\n";
```
For me this happens to work (prints `42`). For you it could work or could crash. If you increase the index ever futher, it will almost certainly crash.

## Catching UB

Luckily C++ has tools that help you catch UB, such as the **address sanitizer** (or **ASAN** for short) and **undefined behavior sanitizer** (**UBSAN** for short).

If you're also following my tooling tutorial, this is included in [the recommended compiler settings](/tooling/articles/recommended_compiler_flags.md) page (`-fsanitize=address`).

Try running the example above with ASAN and observe the error.

I recommend keeping ASAN and UBSAN enabled for the rest of this tutorial. They are not entirely fool-proof, but they help a lot.

The two work better together, as each catches different kinds of UB. The one above happens to be caught by ASAN.
