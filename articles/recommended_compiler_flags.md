# Recommended compiler settings

A compiler (such as Clang) has many different settings you can change.

E.g. if you've read [the page about debugging](/articles/debugging_in_terminal.md), you already know about `-g` that adds debugging information to the executable.

There are many other useful settings. Those settings are also called "flags" or "command-line flags".

This page targets Clang, but most flags here work with GCC too, unless documented otherwise.

Most compiler flags can be used in any order (this is true for all flags on this page).

You can read the [summary](#summary) at the bottom.

## Basic flags you should use by default

Those are the flags I recommend to always use.

Click on each for an explanation.<details><summary><b><code>-Wall -Wextra</code></b> — enable basic compiler warnings</summary>

A "warning" is when a compiler tells you that it thinks your code is bugged, despite being valid C++. It won't tell by default though, you have to ask it.

For example, this program:
```cpp
#include <iostream>

int main()
{
    int sum;
    for (int i = 0; i < 10; i++)
        sum += i;
    std::cout << sum << '\n';
}
```
...compiles but might not work properly because we didn't initialize `sum`. But a compiler will happily compile it. But if you add `-Wall -Wextra` (e.g. `clang++ prog.cpp -Wall -Wextra`), it'll warn you about this:
```none
prog.cpp:7:9: warning: variable 'sum' is uninitialized when used here [-Wuninitialized]
    7 |         sum += i;
      |         ^~~
prog.cpp:5:12: note: initialize the variable 'sum' to silence this warning
    5 |     int sum;
      |            ^
      |             = 0
1 warning generated.
```

**When asking for help with your code, make sure you enabled and fixed the warnings first.**

`W` in `-Wall -Wextra` stands for "warnings". `-Wall` enables the most common warnings (not all of them, despite saying "all"), and `-Wextra` enables some additional warnings (but again not all of them).
</details>

<details><summary><b><code>-pedantic-errors</code></b> — disable problematic language extensions</h3></summary>

Compilers add custom features to C++ that are not described in the standard ("C++ standard" is the document describing how C++ is supposed to work, that all compilers try to follow).

Those extra non-standard features are called "language extensions" or just "extensions".

For example, the following is not legal in standard C++ (unlike C), because array size must be a fixed number:

```cp
#include <iostream>

int main()
{
    int n;
    std::cin >> n;
    int array[n];
}
```

But Clang and GCC accept this by default.

Why is this bad? Because different compilers have different extensions, and if you use them, your program will not compile on some compilers (e.g. MSVC compiler can't compile the program above).

If you try to compile this with `-pedantic-errors` (`clang++ prog.cpp -pedantic-errors`), you should see an error:
```
prog.cpp:7:15: error: variable length arrays in C++ are a Clang extension [-Werror,-Wvla-cxx-extension]
    7 |     int array[n];
      |               ^
prog.cpp:7:15: note: read of non-const variable 'n' is not allowed in a constant expression
prog.cpp:5:9: note: declared here
    5 |     int n;
      |         ^
1 error generated.
```

Some extensions have standard alternatives (e.g. `std::vector<int> array(n);` can replace `int array[n];` above).

Even if you intentionally do want to use an extension, this isn't a good reason to remove this flag. Instead, there are ways to disable those checks for certain *parts* of your code.

#### Alternative flags: `-pedantic` and `-Wpedantic`

Some people use `-pedantic` or `-Wpedantic` instead of `-pedantic-errors`. The first two have the same effect, and unlike `-pedantic-errors` they produce warnings rather than errors.

</details>

<!-- TODO: replace C++26, C23, when newer standards are released -->
<details><summary><b><code>-std=c++26</code></b> — set language version and disable more extensions</h3></summary>

This is for C++, for C use `-std=c23`.

This does two things:

1. It **sets the language version to C++26** (newest supported by Clang at the time of writing). The **language version** is not the same thing as the **compiler version**. The language version is the version of the C++ standard (the document that all compilers try to adhere to, that explains how C++ is supposed to work).

    If you tell a compiler to use an old C++ version, it will disable some new features.

    The number `26` is the release year. At the time of writing, C++26 isn't released yet (but the compilers already support some of the planned features). The latest released C++ standard is C++23, and the past ones were C++20, C++17, C++14, C++11, C++98 (later amended as [C++03](https://stackoverflow.com/q/8285777/2752075)). And the C standards are C23, C17, C11, C98.

    Default language version varies between compilers and compiler versions, at the time of writing Clang and GCC default to C++17.

    You'll see people mention versions with a letter, such as C++2c. It refers to the next version of the standard that's currently in development, and the letter will be replaced with a digit when it's released. C++2c is expected to be released in 2026 and become C++26 (C++2a became C++20, C++2b became C++23; and before that C++0x first became C++1x (they didn't guess the decade right) and then C++11; C++1y became 14 and C++1z became C++17).

2. It **disables some problematic language extensions**.

    A "language extension" is a feature that a compiler adds to C++, that's not mentioned in the C++ standard.

    For example, `int typeof = 42;` is valid in standard C/C++ (`typeof` is a variable name with no special meaning), but it doesn't compile by default in Clang and GCC, because they assign a [non-standard meaning](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html) to `typeof`.

    The default `-std=...` value in Clang and GCC is `-std=gnu++17` and `-std=gnu17` (in C++ and C respectively) (at the time of writing), where "gnu++" stands for "C++ with GNU extensions" (aka GCC extensions, as GCC stands for the "GNU Compiler Collection" or the "GNU C Compiler").

    Replacing `gnu++` with `c++` is what disables those extensions. The code above compiles with any `-std=c++…` flag.

    `-std=c++…` and `-pedantic-errors` work better in tandem. If you only add the latter, `int typeof = 42;` will still not compile, but any use of [`typeof` extension](https://gcc.gnu.org/onlinedocs/gcc/Typeof.html) will not compile either. Adding the former makes `int typeof = 42;` compile.

    You don't lose anything by adding `-std=c++…`, because there are loopholes to use the extensions (e.g. spelling `typeof` as `__typeof` or `__typeof__` will let you use it as the extension; and there's no conflict with variable names because they can't legally contain `__`).

</details>

## Flags to catch errors

Those will catch many programming errors for you. But they decrease performance, and should be disabled in "release builds" (when producing executables to be shipped to end users).

<details><summary><b><code>-fsanitize=address</code></b> — enable address sanitizer</h3></summary>

"Address sanitizer" (or "ASAN" for short) is a tool that catches pointer errors. It's embedded into your executable, and performs additional checks when you run the executable.

Consider this broken program:
```cpp
#include <iostream>

int main()
{
    int arr[5] = {4,5,6,7,8};

    for (int i = 0; i < 10; i++)
        std::cout << arr[i] << '\n';
}
```
Here we access the array out of bounds (read 10 elements while it only has 5).

When I run this, I get following output:
```
4
5
6
7
8
0
1478234208
0
0
0
```
While in most other languages you would immediately get an error, in C++ you get "undefined behavior", meaning anything can happen: you could get an error, but in this case I got 5 random garbage numbers.

ASAN would catch this.

Compile this program with `clang++ prog.cpp -fsanitize=address`, and after `8` you'll get an error message (a fairly cryptic one, but it will say `stack-buffer-overflow`, and that it happened in function `main`).

**If you also add `-g`, it will tell you the exact line number.**.

ASAN also catches memory leaks.

Note that ASAN has a significant performance and memory overhead.

</details>

<details><summary><b><code>-fsanitize=undefined</code></b> — enable undefined behavior sanitizer</h3></summary>

"Undefined behavior sanitizer" (aka UBSAN) catches some forms of undefined behavior.

For example:
```cpp
#include <iostream>

int main()
{
    for (int i = 0; i < 10; i++)
        std::cout << i * 400000000 << '\n';
}
```
If you run, you might get something like this:
```
0
400000000
800000000
1200000000
1600000000
2000000000
-1894967296
-1494967296
-1094967296
-694967296
```
When `i >= 6`, `int` overflows. (In this case it manifests as negative numbers, but in general can cause other issues.)

If we compilie it as `clang++ prog.cpp -fsanitize=undefined` and run, UBSAN will complain:
```
prog.cpp:6:24: runtime error: signed integer overflow: 6 * 400000000 cannot be represented in type 'int'
SUMMARY: UndefinedBehaviorSanitizer: undefined-behavior prog.cpp:6:24
```

</details>

<details><summary><b><code>-D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_DEBUG</code></b> — enable additional checks in the C++ standard library</summary>

For example, this will detect accessing `std::vector` out of bounds.

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v = {1,2,3};
    std::cout << v[10] << '\n';
}
```
Without this flag, this might print a junk number. With the flag, you'll get:
```
C:/msys64/clang64/include/c++/v1/vector:1393: assertion __n < size() failed: vector[] index out of bounds
```

There are also weaker versions of this flag with less overhead, consult [the manual](https://libcxx.llvm.org/Hardening.html).

Note that this flag only works when using libc++ (Clang's own implementation of the C++ standard library.)

When using libstdc++ (GCC's implementation of the C++ standard library) ([which can be used with Clang too](/articles/msys2_environments.md#c-standard-library-implementation)), use **`-D_GLIBCXX_DEBUG`. ([See manual](https://gcc.gnu.org/onlinedocs/libstdc++/manual/using_macros.html). There are some other variations of this flag too.)


</details>

## More warnings

Those are additional warnings that you can and should enable.

<details><summary><b><code>-Wconversion</code></b> — warn on implicit conversions</summary>

This is a rather important warning.

Consider the following code:
```cpp
int main()
{
    float x = 3, y = 5;
    int z = x / y;
}
```
The value of `z` is `0` rather than `0.6`, because its type is not `float`. `-Wconversion` will catch this.

"Implicit conversion" means "without a cast". Adding a cast (`int z = int(x / y);`) disables the warning, because this is now an "explicit conversion", and it expresses that the programmer intended this to happen.

</details>


<details><summary><b><code>-Wimplicit-fallthrough</code></b> — detect forgotten <code>break</code> in <code>switch</code>es</h3></summary>

For example, the following code prints `01` because of the missing `break`s.

```cpp
switch (0)
{
    case 0: std::cout << 0;
    case 1: std::cout << 1;
}
```

The warning will catch this.

In GCC this warning is included in `-Wall -Wextra`, so adding this flag manually is not needed.

</details>

<details><summary><b><code>-Wdeprecated</code></b> — warn when deprecated features are used</h3></summary>

C++ standard declares some features to be "deprecated", meaning they shouldn't be used and eventually might be removed.

One relatively obscure case there this is important is as follows:

```cpp
struct A
{
    std::string x;
    ~A() {}
};

A x;
A y = std::move(x);
```
Here, adding a destructor silently removes move constructor and move assignment, but leaves the copy constructor and copy assignment, which means that `std::move` silently loses its effect and this becomes a copy, which is bad for performance. (You can confirm this by replacing `std::string` with your own class, with logging in copy and move operations.) (If the fields are move-only, such as `std::unique_ptr`, `A` becomes non-copyable and non-movable.)

This catches most people by surprise. `-Wdeprecated` helps here, because the fact that copy operations are not also removed by the presence of a destructor is deprecated, and trying to copy *or move* this class triggers this warning.

GCC seems to also have this flag, but it doesn't catch the problem above.

</details>

<details><summary><b><code>-Wextra-semi</code></b> — warn on excessive <code>;</code>s</h3></summary>

This warns when semicolons are used unnecessarily. This is legal and doesn't cause any issues, but looks uncool.

Example:

```cpp
int main()
{

};
```

GCC also has this warning, but it works in less contexts.

</details>

## Optimization options

"Optimization" refers to improving certain properties of a program, usually performance. Either manually, or in this case automatically by the compiler.

<details><summary><b><code>-O3</code></b> — optimize performance</h3></summary>

This happens at the cost of increased compilation time. It can also break programs containing undefined behavior.

`-O2` and `-O1` are the weaker variants of this.

Somewhat interfers with debugging (with `-g`).

`-Og` is a weaker version designed to play well with `-g`.

</details>

<details><summary><b><code>-flto=thin</code></b> — enable link-time optimization</h3></summary>

This improves optimization of multifile programs, at the cost of increased link time.

There's another version of this, `-flto`, which is slightly stronger, but makes the link time even slower. (GCC only has non-thin `-flto`.)

</details>

<details><summary><b><code>-s</code></b> — strip the resulting executable</h3></summary>

[Stripping](https://en.wikipedia.org/wiki/Strip_(Unix)) an executable sligtly decreases its size and makes the debugging harder.

Not compatible with `-g`, as it would remove debugging information.

</details>

<details><summary><b><code>-DNDEBUG</code></b> — disable <code>assert()</code> macro</h3></summary>

If you're using [`assert()`](https://en.cppreference.com/w/cpp/error/assert) in your code, this disables it. It's intended to only be enabled during development.

</details>

## Summary

Summarizing the above, I recommend following compilation flags:

**`-std=c++23 -pedantic-errors -Wall -Wextra -Wconversion -Wimplicit-fallthrough -Wdeprecated -Wextra-semi`**

And additionally:

* In debug builds (i.e. during development):<br/>
  **`-g -D_LIBCPP_HARDENING_MODE=_LIBCPP_HARDENING_MODE_DEBUG -fsanitize=address -fsanitize=undefined`**

  If you don't get enough performance, add `-Og` and/or disable `-fsanitize=...`.

* In release builds (i.e. final executables for end users):<br/>
  **`-O3 -s -flto=thin`**
