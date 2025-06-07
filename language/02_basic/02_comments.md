# Comments

Large and complex programs can be hard to read, so programmers tend to add explanations to them to help other programmers and future selves. Those explanations are called **comments**.

Comments are ignored by the compiler and can contain any text. For example:

```cpp
// This is a simple test program.

#include <iostream>

int main() // This is the main function!
{
    // This line prints some text.
    std::cout << "Hello, world!\n";
}
```
Anything from `//` to the end of the line is a comment.

Note that this example is a bit excessive. Don't explain the things that are obvious. Only explain things that might otherwise confuse your future self or others.


It's customary to put a comment on the line *before* the thing it's explaining (or on the same line after it).

There's also another style of comments: `/* ... */`. Those end not at the end of the line, but at the matching `*/`, and may or may not span multiple lines. For example:
```cpp
#include <iostream>

int main()
{
    /* This code prints
    some text. */
    std::cout << /*we print this string -> */ "Hello, world!\n";
}
```
This is once again an excessive example, you probably shouldn't write comments like this.

It's customary to use `/* ... */` for long comments spanning many lines, and `//` for short comments consisting of one or a few lines at most.
