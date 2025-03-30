# Why do I recommend Clangd over the official C/C++ plugin?

I prefer Clangd to the official C/C++ plugin, but you can use whatever you like more.

## Availability

Clangd is fully open-source, and plugins for it exist for most popular IDEs.

The official C/C++ plugin is partially closed-source (notably the intellisense itself and the debugger appear to be closed-source), and only works with VSC (and VS presumably uses the same intellisense engine).

## Performance

In my experience, Clangd has a slightly faster repronse time.

The official C/C++ plugin needs a full second to update the error squiggles after changing the code (even in a hello world program), while Clangd does it instantly. (Though this was tested)

## Configuration complexity

Both understand the [`compile_commands.json`](https://clang.llvm.org/docs/JSONCompilationDatabase.html) file, which is probably what you'll be using most of the time.

Both have their own additional configuration files in their own formats, but I don't see a big difference in complexity between the two.

## Extra features

Clangd provides just the code completion, while the C/C++ plugin provides some extra features (such as debugging support).

You can either install additional extensions for those (like [LLDB-DAP](/tooling/articles/configuring_vsc_debugger.md) for debugging), or keep the C/C++ plugin installed at the same time to provide them. (I'm not aware of any important missing features other than debugging.)

## Accuracy

Clangd has a big benefit of *never* disagreeing with Clang the compiler, and not lagging behind on C/C++ feature support (while it might not *suggest* certain new features it doesn't fully understand, it will at least not mark them as errors).

The official C/C++ plugin's intellisense isn't based on an actual compiler (but rather on [this](https://www.edg.com/c)), so there's no matching compiler that it always agrees with.

The official C/C++ plugin chokes on certain language features (albeit obscure ones), notably on stateful metaprogramming: (same as VS intellisense)
```cpp
struct Reader
{
    friend auto foo(Reader);
};
template <typename T>
struct Writer
{
    friend auto foo(Reader) {return T{};}
};

int main()
{
    (void)Writer<int *>{};
    int *p = foo(Reader{}); // C/C++ plugin marks this as an error.
}
```
