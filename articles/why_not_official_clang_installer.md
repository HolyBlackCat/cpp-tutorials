# Why do I not recommend using the official Clang installer with MinGW?

On Windows, Clang can be installed in different ways. There is [the official installer](https://github.com/llvm/llvm-project/releases) (look for `LLVM-<version>-win64.exe`), and several unofficial distributions with minor modifications.

In particular, MSYS2 provides its own Clang.

## Difference between the Clang distributions

Clang on Windows is not self-sufficient, at least not the one from the official installer.

It needs another compiler to be installed to use its standard library and various tools distributed with it. Clang can use [either an MSVC installation, or a MinGW one](/articles/choosing_compiler_and_more.md).

Clang from the official installer looks for MSVC by default, but you can make it look for MinGW instead (using e.g. the `--target=x86_64-w64-mingw32` flag). Clang from MSYS2 looks for MinGW by default. The default flags are the only difference ([you can review the differences here](https://github.com/msys2/MINGW-packages/tree/master/mingw-w64-llvm)).

In the past, when Clang was less mature and had problems with MinGW compatibility, MSYS2 distributed a patched version with improved compatibility. This doesn't seem to be necessary anymore, and all remaining patches only affect default flags.

## Conclusion

If you're planning to use Clang with MSVC, the official installer might be a better choice.

If you're planning to use it with MSYS2 MinGW, using their version of Clang will make your life a bit easier.

All of this applies to [Clangd](/articles/configuring_code_completion.md) as well.
