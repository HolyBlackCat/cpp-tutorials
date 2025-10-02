# Comments

Large and complex programs can be hard to read, so programmers add explanations
to help other programmers and themselves, called **comments**.

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
Anything from `//` to the end of the line is a line comment, because it takes up
one line.

Note that this example is a bit excessive. These comments don't explain anything
that isn't already obvious. Use comments to explain code that might be confusing.

We usually put line comments on the line *before*, or sometimes on the same line
*after* what we want to explain.

There are also block comments: `/* ... */`. Block comments begin with `/*` and
end with `*/`, even if it's not on the same line:
```cpp
#include <iostream>

int main()
{
    /* A long comment explaining many important details about how some part of
    the code works, what it does and why it does it. It can to be very long
    if the code is very complex or very confusing. */
    std::cout << /* An inline comment */ "Hello, world!\n";
}
```
