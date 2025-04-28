# Strings

You already know how to print strings, now is the time to learn more about how they work.

## String variables

To create a string variable, you `#include <string>` and then use the `std::string` type:
```cpp
std::string my_name = "Egor";
```

For most purposes `std::string` acts like a `std::vector`. But of what element type?

If you try printing `my_name[0]`, `my_name[1]`, etc, you'll see that it prints the individual characters.

Those elements have type **`char`**:
```cpp
std::string my_name = "Egor";
char first_letter = my_name[0];
std::cout << first_letter << "\n"; // Prints `E`.
```
There are `char` literals too, which use `'` instead of `"`:
```cpp
char letter = 'A';
std::cout << letter << "\n"; // A
```
In comparison, `char letter = "A";` is a compilation error.

Single quotes `'...'` should contain exactly **one** character. Empty `''` is a compilation error. Having multiple characters in it is technically allowed, but what value that returns depends on the compiler, so it should be avoided.

Note that `'\n'` is allowed, because `\n` counts as one character.


`char` acts very similar to `int`. It's also an integer, but with a smaller range of values: typically -128...127 or 0...255 depending on the compiler and its settings, whereas `int` can typically hold numbers between around ±2000000000 (more on those limits will be explained in future chapters).

Different letters/symbols are represented using different numbers:

```cpp
char letter = 'A';
std::cout << letter << "\n"; // A
int code = letter; // Note implicit conversion from `char` to `int`.
std::cout << code << "\n"; // 65
letter = 66; // Note implicit conversion from `int` to `char`.
std::cout << letter << "\n"; // B
std::cout << char(67) << "\n"; // C, note the cast from `int` to `char`.
std::cout << int('C') << "\n"; // 67, note the cast from `char` to `int`.
```

For English letters and common punctuation, those numbers are

`char` holds the code
