# Why MinGW?

This is a TL;DR of [Choosing a compiler and more](./choosing_compiler_and_more.md), see that for more details. Also see [Why not visual studio?](./why_not_visual_studio.md).

I don't recommend MSVC due to it being [wonky and buggy in general](./choosing_compiler_and_more.md#msvc-issues), which leaves GCC and Clang. Clang then wins because of numerous [quality-of-life features](./choosing_compiler_and_more.md#choosing-a-compiler) (sanitizers on Windows, easier cross-compilation, Clangd existing, etc).

Clang on Windows can be used either in MSVC-compatible mode or MinGW-compatible, which isn't ultimately that important.

MSVC ABI [is slightly more jank](./choosing_compiler_and_more.md#mingw-abi-vs-msvc-abi), such as requiring `[[msvc::no_unique_address]]`, empty base optimization not working in some cases, etc. Also MSVC STL has [some annoying properties](./choosing_compiler_and_more.md#choosing-c-standard-library), such as `std::deque` being slow, some iterators not being const-correct, `std::async` being non-conformant. Technically MSVC ABI doesn't require MSVC STL, you can also use libc++; but I'm not aware of prebuilt libc++ binaries using MSVC ABI for windows.

But since MSVC ABI is more popular, using it could be required for compatibility with existing libraries not supporting MinGW.
