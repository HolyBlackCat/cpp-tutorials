# Strings

You already know how to print strings, now is the time to learn more about how they work.

## String variables

To create a string variable, you `#include <string>` and then use the `std::string` type:
```cpp
std::string my_name = "Egor";
```

For most purposes `std::string` acts like a `std::vector` (it has `[]`, `.size()`, defaults to being empty rather than uninitialized, etc).

But what's its element type?

If you try printing `my_name[0]`, `my_name[1]`, etc, you'll see that it prints the individual characters.

Those elements have type **`char`**:
```cpp
std::string my_name = "Egor";
char first_letter = my_name[0];
std::cout << first_letter << "\n"; // Prints `E`.
```
So `std::string` behaves similarly to `std::vector<char>`.

There are `char` literals too, which use `'` instead of `"`:
```cpp
char letter = 'A';
std::cout << letter << "\n"; // A
```
In comparison, `char letter = "A";` is a compilation error.

Single quotes `'...'` should contain exactly **one** character. Empty `''` is a compilation error. Having multiple characters in it is technically allowed, but what value that produces depends on the compiler, so it should be avoided.

Note that `'\n'` is allowed, because `\n` counts as one character. (So far all examples were using `"\n"` to postpone explaining what `char` is, but I'm going to switch to `'\n'` now, since it's likely a bit faster to print.).


`char` acts very similar to `int`. It's also an integer, but with a smaller range of allowed values: typically -128...127 or 0...255 depending on the compiler and its settings, whereas `int` can typically hold numbers between around ±2000000000 (more on those limits will be explained in later chapters).

Different letters/symbols are represented using different numbers, and you can use conversions between `char` and `int` to inspect those numbers:

```cpp
char letter = 'A';
std::cout << letter << '\n'; // A
int code = letter; // Note implicit conversion from `char` to `int`.
std::cout << code << '\n'; // 65
letter = 66; // Note implicit conversion from `int` to `char`.
std::cout << letter << '\n'; // B
std::cout << char(67) << '\n'; // C, note the cast from `int` to `char`.
std::cout << int('C') << '\n'; // 67, note the cast from `char` to `int`.
```

While the C++ itself doesn't guarantee which numbers correspond to which letters, it most often follows the **ASCII encoding** ("encoding" being a system that assigns numbers to specific characters) that is used almost everywhere today, in one form or another.

ASCII only describes English letters and basic punctuation. The situation with other languages is more difficult and is explained later in this chapter.

## Comparing characters

Operators like `<` and `>` do work on `char`s. And since, for example, digits (from 0 to 9) have consecutive codes (check the ASCII chart), `x >= '0' && x <= '9'` is a valid way to test if `x` is a digit character. Similarly, `x >= 'a' && x <= 'z'` for lowercase letters, and similarly for the uppercase ones.

## Inputting strings

`std::cin >>` can be used to input `std::string`s, just like numbers.

But if you try using it, you'll quickly notice that `std::cin >>` stops at whitespace, so e.g. `Hello, world!` would be read as two separate strings, `Hello,` and `world!`.

To read the entire line (possibly with spaces), do this:
```cpp
std::string s;
std::getline(std::cin, s);
```

> ## Exercise 1
>
> Make a program that asks the user for their name, and then prints `Hello` and that name.
>
> Try doing that using both `>>` and using `getline`. Try inputting a name with and without spaces into both programs and see what happens.

### Mixing `getline` and `>>`

There is a common pitfall that occurs when mixing `getline` with `>>`:
```cpp
int x;
std::cin >> x;
std::cout << x << '\n';

std::string s;
std::getline(std::cin, s);
std::cout << s << '\n';
```
If you try to input:
```console
42
Hello
```
This prints `42` and nothing instead of `Hello`. (It doesn't even let you input `Hello` before closing the program.)

This is because `>>` consumes `42`, but not the newline character after it. Then `getline` consumes the rest of the line until the `\n`, which is empty. Adding yet another `getline` after that would read the correct string.

You can see this more clearly by inputting `42 Hello`, which prints `42` and ` Hello` (with a space before `H`).

And the reason why multiple subsequent `>>`s are not affected by this issue is that `>>` skips any whitespace before reading the value, and newlines count as whitespace too.

A simple solution to this is to do `getline` twice. Or perhaps after doing it the first time, do it again if the string was empty.

> ## Exercise 2
>
> Make a program that asks you for your age, and then your name, and then prints both.
>
> Make sure spaces in the name are handled correctly.
>
> Make sure inputting the name works both on a separate line and on the same name as the age.

## Operations on strings

Here are some things you can do to strings:

### Getting substrings

```cpp
std::string s = "Hello!";
std::string part = s.substr(2);
std::cout << part << '\n'; // Prints `llo!`.
```
`s.substr(n)` returns a part of the string, skipping the first `n` characters.

```cpp
std::string s = "Hello!";
std::string part = s.substr(2, 3);
std::cout << part << '\n'; // Prints `llo`.
```
`s.substr(n, m)` returns a part of the string, skipping the first `n` characters and taking only `m` characters after that.e

### Combining strings

`+` can be used to combine ("concatenate") strings:

```cpp
std::string name;
std::getline(std::cin, name);
std::string message = "Hello " + name + ", nice to meet you!";
std::cout << message << '\n';
```

### Converting numbers to strings

You can't directly convert a number to a string. Instead, use `std::to_string`:
```cpp
int x = 42;
std::string s = std::to_string(x);
std::cout << s << '\n'; // 42
```
Notice that you can't use `<<` to produce complex strings. You can do:
```cpp
int health = 42;
std::string message = "You have " + std::to_string(health) + " HP";
```
...but can't do:
```cpp
std::string message "You have " << health << "HP";
```
...because it's `std::cout` that makes `<<` work this way. It *is* possible to do this (using `std::ostringstream`, which is the equivalent of `std::cout` that "prints" to a string instead of to the screen), but I'm not going to explain it here.

And if you forget `std::to_string`, you might observe some funny effects:
```cpp
std::cout << "The value is " + 5 << '\n';
```
This prints `lue is ` (trims the first 5 characters of the string). This of course shouldn't be done intentionally, and will be explained in a later chapter.

### Coverting strings to numbers

The opposite conversion, from a string to a number, can't be done directly too. There are many ways to do it, and `std::stoi` and `std::stod` (for `int` and `double` respectively):
```cpp
std::string s = "42";
int n = std::stoi(s);
std::cout << n << '\n'; // 42
```
Note that `std::stoi` and `std::stod` silently skip any whitespace at the beginning of the string, and also ignore any non-digits after the number. I'm not going to explain how to fix it here, look this up if you need it.

> ## Exercise 3
>
> Write a program that inputs the string, finds every number in it, and for each number prints that many `*` characters on a separate string. For example, `You have 10 apples and 5 peaches` should result in:
> ```console
> **********
> *****
> ```
> You could use a second `string` to collect the digits when looping over the first one, and then pass them to `std::stoi`. Or use some other approach.

## String literals

As was explained before, unnamed constants in the source code such as `42` or `"Hello"` are called literals (in this case, an integer literal and a string literal).

Since `42` is an `int`, you'd naturally assume that `"Hello"` is a `std::string`, but that's not the case.

It is in fact a constant array of `char`, which is something C++ inherited from C.

Since it's an array rather than a `std::string`, `"Hello".size()` is illegal. But `"Hello"[0]` is legal and produces `'H'`.

Since the array is constant, `"Hello"[0] = 'A';` is a compilation error.

### Initializing arrays with string literals

Normally you can't initialize one array with another, but string literals and `char` arrays are a special case:

```cpp
char str[] = "Hello";
```
This is legal and is equivalent to `char str[] = {'H', 'e', 'l', ...};`

## Null terminators

If you check the array size, which can be done like this:
```cpp
#include <iostream>
#include <iterator> // For `std::size`.

int main()
{
    std::cout << std::size("Hello") << '\n'; // 6

    char str[] = "Hello";
    std::cout << std::size(str) << '\n'; // 6
}
```
...you'll see that `"Hello"` and `str` arrays both contains `6` elements, rather than `5` as you would expect.

The last element is a zero character, also called a **null terminator** (null = zero, terminator = it terminates (ends) the string). The null terminator is `0`, not `'0'`. Do you understand the difference?

`'0'` is a character like any other (like e.g. `'A'`). If you print `int('0')`, you'll see that its ASCII code is `48`. Whereas the null terminator has ASCII code `0`.

The null terminator is a non-printable character, it doesn't correspond to some symbol that can be printed. If you try printing it (print `char(0)`), you'll likely see nothing.

The null terminator is often spelled as `'\0'`. That is equivalent to `char(0)`, and **not** equivalent to `'0'`, as was explained above.

Null-terminated strings are usually called **C-strings**, since the C language uses them a lot.

### Why null terminators?

They are used to loop over strings without knowing their size. For example:
```cpp
char str[] = "Hello";
for (int i = 0; str[i]; i++)
    std::cout << str[i] << '\n';
```
This prints every character individually. Notice `str[i]` being used as a condition, it tests that `str[i]` is not a null terminator, and stops the loop when encountering one. The `str[i]` condition is equivalent to `str[i] != 0` or `str[i] != '\0'`.

Now, you might be wondering how is this better than using `i < 5` or `i < std::size(str) - 1`, and the answer is that it is not better. It can be useful in some other situations where you don't know the array size, which will make sense in the later chapters, but in the cases you've already seen it is indeed useless.


### Null-terminators in different places

`"Hello"` is null-terminated.

`char str[] = "Hello";` is also null-terminated and is equivalent to:
```cpp
char str[] = {'H', 'e', 'l', 'l', 'o', '\0'};
```
But this is not null-terminated:
```cpp
char str[] = {'H', 'e', 'l', 'l', 'o'};
```
Non-`char` arrays are also not null-terminated unless you do that manually.

`std::string`s **are** always null-terminated, but unlike `char` arrays, the size they report doesn't include the terminator.

E.g. for `std::string s = "Hello";`, `s.size()` is `5` (and so is `std::size(s)`, which is the same thing as `s.size()` for everything other than arrays, which don't have `.size()`). And `s[5]` is legal and produces `'\0'`.

Whereas arrays and vectors don't allow using their size as the index (the valid indices go up to the size minus 1). So, for `char s[] = "Hello";`, `std::size(s)` is `6`, `s[5]` is `'\0'`, and `s[6]` is illegal (causes UB).

### What requires null terminators?

If you're using `char` arrays instead of `std::string`s (there's little reason to), you'll notice that some standard library features require them to be null-terminated.

For example:
```cpp
char str[] = {'H','e','l','l','o'}; // Not null-terminated.
std::cout << str << '\n'; // Causes UB!
```
First, notice that `std::cout` can print `char` arrays as a special case (it can't print whole arrays of most other types).

This code causes undefined behavior, because `std::cout` wants `char` arrays to be null-terminated. If you have ASAN enabled, it will complain about this. If not, you'll likely see some garbage characters printed after your string.

> ## Exercise 4
>
> Using a cast to `int` to print character codes, print the code of a null terminator and observe that it's 0.
>
> Make a non-null-terminated char array, try printing it with `cout` and try observing the breakage (which may or may not happen, since it's UB). Now fix it by adding the null terminator.

## Languages other than English

While English strings "just work", as soon as you start adding letters from other languages, or unusual characters such as emojis, you can get issues if you don't know what you're doing.

You can skip this section if you don't care about those for now.

As was mentioned above, `char` can only hold 256 different values (either -128...127 or 0...255). But there are clearly more different characters in all the languages combined, not to mention emojis and such. There are several solutions to this:

1. You can represent the rare characters as sequences of **multiple** `char`s. This is what the civilized programming world is doing. The most popular system of doing so (encoding) is called **UTF-8**.

2. You can continue using one `char` per character, but vary the encoding depending on the current system language. This works if you only need the letters from English and from your one language (if combined there are no more than 256 of them), but this easily breaks if you run your program on a machine that has a different language selected, unless you take special care to avoid that breakage. And most serious programs need to be able to handle more than one language at the same time anyway.

3. You can abandon `char` altogether and use a larger type (with a bigger range), such as `wchar_t` ("wide" character) which on Windows has range 0...65535 (see UTF-16). This sounds good in theory, until you end up needing even more range, and suddenly have to use sequences of *several* `wchar_t`s per character (with the same problems as in (1)). Using an even wider type is possible (with range 0...4294967295, see UTF-32), but there are still certain characters that are represented with *several* of those values (not because we ran out of range, but this time simply by convention).

   So ultimately chasing larger types seems futile, because you still end up with multiple values per character, and have to handle sequences of values in a special manner.


All of those are extensions of ASCII. The number 0...127 (which is what ASCII covers) have the same meaning in all three. Only the remaining values have different meanings.

The civilized programming world uses (1), that is UTF-8. It is the default on Linux and Mac, and basically everywhere except Windows.

Windows primarily uses (3), but also (2). But you can force it to use (1) as well.

I recommend embracing option (1). To do that:

1. Keep in mind that a single `char` might not correspond to a single character.

   If you're just inputting and printing strings, you don't need to worry about this, but if e.g. if you want to print one character per line from a given string, you can't use a simple loop like the earlier example in this chapter. You have to determine which groups of `char`s correspond to one character, and print one group per line.

   This is a complicated topic, not for this tutorial. Some simple operations can be done by manually doing math on `char`s. More complex operations are easier to do with libraries (there's not much in the standard library for this, you'll have to install additional ones).

2. On Windows, you need to explicitly enable UTF-8 (to get behavior (1) as opposed to (2)) by doing certain things at the beginning of `main`.

    I'm giving this without explanation for now, and hope to explain some parts in more detail in the later chapters.

    ```cpp
    #include <iostream>

    // Including `windows.h` to get access to `SetConsoleCP()` and `SetConsoleOutputCP()`.
    // What's going on here will be explained in more detail in later chapters.
    #ifdef _WIN32
    #define WIN32_LEAN_AND_MEAN
    #define NOMINMAX 1
    #include <windows.h>
    #endif

    int main()
    {
        // Enable UTF-8.
        #ifdef _WIN32
        SetConsoleCP(CP_UTF8);
        SetConsoleOutputCP(CP_UTF8);
        #endif

        std::cout << "Привет!\n";
    }
    ```

3. If you have any string literals with non-English letters like the above, ensure that your IDE saves the file in the UTF-8 encoding. E.g. Visual Studio Code displays the current encoding in the bottom-right corner.

4. If you're using the MSVC compiler, enable the `/utf-8` setting.

Doing steps 2,3,4 above should hopefully make the example above work correctly.

> ## Exercise 5
>
> Skip this if you skipped the section above about non-English characters.
>
> Write a program that prints a non-English string. Compare how it works with and without the workaround provided in the previous section.
>
> Print the size of the string and confirm that it's larger than the apparent number of characters in it. Print the values of the individual `char`s by casting them to `int`, observe multiple integers per character.
