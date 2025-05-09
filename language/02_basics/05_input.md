# Taking input

I need to explain more about variables, but let's digress for a moment.

So far all programs you've made weren't **interactive**, meaning they always did the same thing when ran, without letting you interact with them (do something to them to affect their behavior).

Let's change that. Here's a small program that adds together two numbers:

```cpp
#include <iostream>

int main()
{
    int x = 0, y = 0;
    std::cin >> x;
    std::cin >> y;
    std::cout << x + y << "\n";
}
```

First of all, notice that we've declared two variables in the same declaration. This is entirely equivalent to `int x = 0; int y = 0;`, and most people prefer to declare one variable per declaration.

If you run this, you'll see nothing, which is to be expected. Type two numbers, e.g. `10 20` and press <kbd>Enter</kbd>, and you should see their sum being printed (`30` in this case).

When the program reaches `std::cin >> ...;`, it pauses and waits for you to type something. And then it assigns what you typed to that variable. The variables get overwritten, so the original values (`= 0`) don't really matter here.

`std::cin` is short for "character input", and also comes from `#include <iostream>`, like `std::cout`. It lets you **input** things (meaning take information from the user of the program).

Just like `std::cout` is often called "the output stream", `std::cin` is called "the input stream", hence the name `<iostream>`.

## `<<` and `>>`

Notice that `cout` uses `<<` while `cin` uses `>>`. This makes sense intuitively, as the information flows from the user **into `>>`** your variable, and to the output **from `<<`** your expression.

### `<<` and `>>` are not symmetric

You can't do `x << std::cin;` or `x >> std::cout;`. That's simply how it is.

### Multiple `>>` per statement

Just like you can do `std::cout << 1 << 2;`, you can also do `std::cin >> x >> y;` in one line of code (one statement), as opposed to `std::cin >> x; std::cin >> y;`.

The two are completely equivalent, just like with `<<` and `std::cout`.

## Asking nicely for input

You'll often see code like this:
```cpp
std::cout << "Please input x: ";
int x;
std::cin >> x;
```
Here the program tells you what it wants you to input, instead of just waiting on an empty screen.

Also notice how we don't have to add `\n` here. This will make the input appear on the same line after the printed text. Whether or not to add `\n` is of course up to you.

## Pausing for input

I previously said that `std::cin >> x;` pauses the program to wait for input.

Yet when we input *two* variables (see the top of the page), we only had to press the <kbd>Enter</kbd> once. Why?

Well, you could press it twice. `10` <kbd>Enter</kbd> `20` <kbd>Enter</kbd> also works and has the same effeect as `10 20` <kbd>Enter</kbd>.

When you input **more** things than necessary, `std::cin` leaves them for later. So later when you use `std::cin >>` again, it now **doesn't** wait for input and instead consumes those.

You can observe this better by doing this:

```cpp
#include <iostream>

int main()
{
    int x;
    int y;
    int z;
    std::cout << "Input x: ";
    std::cin >> x;
    std::cout << "Input y: ";
    std::cin >> y;
    std::cout << "Input z: ";
    std::cin >> z;
    std::cout << x + y + z << "\n";
}
```
If you do `10` <kbd>Enter</kbd> `20` <kbd>Enter</kbd> `30` <kbd>Enter</kbd>, it will show up as:
```
Input x: 10
Input y: 20
Input z: 30
```
Try running this program and see for yourself.

But if you do: `10 20 30` <kbd>Enter</kbd>, you'll see just this:
```
Input x: 10 20 30
Input y:
Input z:
```
...and the program won't pause after `Input y:` and `Input z:` now.

That is because you provided more input than necesasry the first time, so it didn't have to wait for more.

Try to experiment with this a bit. E.g. input 2 numbers instead of 3 and see what happens.

## Inputting wrong things

Try to type something other than a number, e.g. `blah`, and see what happens.

When `std::cin` fails to read what you asked it to read, it should assign zero to that variable, and go into an "error state". Meaning the program is unpaused, and all future `std::cin >> ...;` statements are ignored.

It's possible to recover from this state (to e.g. to ask the user for new input), but I'm not going to explain this now.

## Exercise

Make a program that converts days to minutes. Ask the user for the number of days, and print the number of minutes in those.
