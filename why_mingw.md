# Why do I recommend MinGW over MSVC?

First of all, the choice of the compiler is a religious matter. :P

I tried to explain my reasoning below, but you can use whatever works best for you.

## What choices you can make?

### Available compilers

The three most popular compilers are:

* **MSVC** (stands for "**M**icro**S**oft **V**isual **C**++"), the compiler bundled with Visual Studio.
* **GCC**, the compiler commonly used on Linux.

  On Windows it's usually bundled with something called [**MinGW**](https://www.mingw-w64.org/) (a collection of tools and libraries used to compile Windows applications without Visual Studio).

* **Clang**, the most recent addition to the list ("recent" as in "first release was in this millenium").

But wait, there's more. Clang can operate in different configurations:

&nbsp;|[Compiler](#choosing-a-compiler)|[ABI and various standard libraries](#mingw-abi-vs-msvc-abi)|[C++ standard library](#choosing-c-standard-library)|Available sanitizers|Recommended way to install
---|---|---|---|---|---
1|MSVC|MSVC|MSVC STL|ASAN|With [Visual Studio](https://visualstudio.microsoft.com), or standalone ["build tools"](https://visualstudio.microsoft.com/downloads).
2|Clang|MSVC|MSVC STL|ASAN**|[Official Clang installer](https://github.com/llvm/llvm-project/releases/tag/llvmorg-18.1.6) (look for `LLVM-...-win64.exe`), also must install MSVC.
3|Clang|MSVC|Clang's libc++|ASAN, UBSAN|[Must compile libc++ manually.](https://libcxx.llvm.org/BuildingLibcxx.html)
4|Clang|MinGW|Clang's libc++|ASAN, UBSAN|With [MSYS2 CLANG64](TODO_MSYS2_ENVS) (other [MSYS2 environments](TODO_MSYS2_ENVS) have it too but without sanitizers).
5|Clang|MinGW|GCC's libstdc++|None*|With [MSYS2 UCRT64 or MINGW64](TODO_MSYS2_ENVS). Could also use official Clang installer with any MinGW, but that requires some custom command-line flags.
6|GCC|MinGW|GCC's libstdc++|None*|With [MSYS2 UCRT64 or MINGW64](TODO_MSYS2_ENVS). There are some other distributions too.

As you can see, Clang on Windows can use either MinGW **OR** tools and libraries shipped with MSVC.

You can use any IDE with any of those combinations, with more or less effort (e.g. using Visual Studio while compiling for MinGW might cause [intellisense](https://learn.microsoft.com/en-us/visualstudio/ide/using-intellisense?view=vs-2022) to misbehave).

**"ABI"** stands for ["application binary interface"](https://en.wikipedia.org/wiki/Application_binary_interface). Loosely speaking, if two compilers use the same ABI, you can use any library compiled with one compiler with another. Some C (not C++) libraries might be compatible regardless.

**"Sanitizers"** are automatic tools that you can enable to help you catch bugs: ASAN ("Address Sanitizer") for pointer errors, UBSAN for some other undefined behavior. (Note that on Linux, both GCC and Clang support both ASAN and UBSAN, plus some sanitizers not available on Windows.)

<sup>* - You can UBSAN, but [in a weird mode](https://stackoverflow.com/a/59083808/2752075) that doesn't display errors and just crashes when a violation is detected.</sup><br/>
<sup>** - Should work, but didn't immediately work out of the box for me.</sup>

## Choosing a compiler

All three are viable, but here's my personal order or preference:

1. Clang - It's nice to have support for different C++ standard libraries and ABIs. For some platforms it's the only compiler (e.g. Emscripten, Android NDK).

   Most code analysis tools (including those in most IDEs, excluding VS) are based on Clang today, so you'll likely have to keep your code compatible with it regardless.

2. GCC - Ok, but slower compilation speed compared to Clang on Windows, and worse memory usage (about 2x more). Cross-compilation* requires slightly more effort**. No sanitizers on Windows.
3. MSVC - [Has some issues with optimization quality and standard conformance.](#msvc-issues) Slow at adding support for C++23 and C++26. Not open-source. Cross-compilation is impractical***. But compilation speed is good (same as Clang's or a bit faster).

   Has Address Sanitizer on Windows, which is better than GCC (which got nothing on Windows) but worse than Clang (which also got UB Sanitizer on Windows).

<sup>* - Cross-compilation is compiling for one operating system while using another. Such as compiling for Windows while using Linux.</sup><br/>
<sup>** - While the same Clang executable can compile for any platform (if you provide it with the right libraries), GCC needs to be recompiled for each target platform, which takes time.</sup><br/>
<sup>*** - While you can [run MSVC in Wine](https://github.com/mstorsjo/msvc-wine), it's quite slow for most practical purposes.</sup>

## Choosing C++ standard library

There's no single "C++ standard library". Rather, there are several implementations of the same specification:

* Clang's **libc++**

  Slightly less mature than the other two, some minor features are missing as of libc++-18, such as: parallel standard algorithms, iteration validation.

  They seem to have made some clever design choices, such as a very memory-efficient SSO for `std::string`s.

* **MSVC STL**

  Has some minor warts that they don't fix to preserve ABI compatibility, such as: [`std::deque` being a joke](https://github.com/microsoft/STL/issues/147), or `std::async` non-conformingly using a thread pool, which messes with `thread_local` variables.

  Otherwise seems to be quite decent.

* GCC's **libstdc++**

  The golden standard. Most Linux distributions use it by default.

If you're using GCC or MSVC, you're locked into their respective C++ standard libraries, but Clang can use any of the three.

On MinGW I prefer libstdc++ rather than libc++ to have iterator validation (TODO compiler flags), but on the other hand libc++ lets you use the address sanitizer. So it's a good idea to test on both.

## MinGW ABI vs MSVC ABI

MinGW and MSVC are incompatible. They use different standard libraries. Libraries compiled with one can't be used with another (except perhaps for some C (not C++) libraries).

If you're using GCC or MSVC, you have no choice.

Clang be switched between MSVC mode and MinGW mode:

* If you're using either libstdc++ or MSVC STL, that locks you into the respective mode.

* If you have prebuilt libraries compiled for one of those two ABIs, and you can't compile them for another (e.g. because they are closed-source), that locks you into that ABI.

* MSVC ABI has quirks, such as `[[no_unique_address]]` not working out of the box (must use `[[msvc::no_unique_address]]`).

MinGW is less popular/has less inertia behind it, which means in very rare cases some libraries or applications may not support it. Sometimes you can patch the libraries, but sometimes not. Some libraries do the opposite and only support MinGW.

## MSVC issues

Elaborating on the [claims](#choosing-a-compiler) I made above:

* "Issues with standard conformance" - They are getting better at this, but still have some sloppy behavior, such as:

  * `[[no_unique_address]]` having no effect (have to use `[[msvc::no_unique_address]]`).
  * Accepting `typename` in wrong locations (`typename int x;`).
  * [Accepting incorrect `requires`-clauses.](https://stackoverflow.com/q/78155107/2752075)
  * ...

* Not implementing C++23 and C++26 features. If you take a look at [cppreference](https://en.cppreference.com/w/cpp/compiler_support/23), at the time of writing, they didn't implement any core language features other than "deducing this" (and a few minor defect reports/tweaks). Note that I'm talking only about the core language features, not standard library features.

  Historically, they were slow to adopt C++11, then slowly started to overtake GCC and Clang up to C++20, and now they are behind again.

* Has some weird bugs/quirks, the kind that you don't see with other compilers: [[1]](https://stackoverflow.com/q/78157781/2752075), [[2]](https://stackoverflow.com/q/69431575/2752075).
  <details>
  <summary>Or e.g. this one.</summary>

  ```cpp
  int main()
  {
      constexpr bool a = false;

      auto lambda = []
      {
      label:
          [[maybe_unused]] constexpr bool a = true;
          static_assert(a); // Fails with `/std:c++latest`. Stops failing if you remove either the label or remove the attribute.
      };
  }
  ```
  </details>

* "Issues with optimization quality" - I've personally had some minor problems, but other sources seem to confirm this: [[1]](https://www.agner.org/optimize/optimizing_cpp.pdf#page=10), [[2]](https://lemire.me/blog/2023/02/27/visual-studio-versus-clangcl/), etc.

* Not cross-platform:

  * Can only compile for Windows.
  * Can only be used on Windows (not counting weird setups using Wine), while GCC and Clang can cross-compile from other operating systems.

* Not open-source:

  Most of time it's just a matter of ideology, but it means you can't fix bugs in it yourself.

  Note that MSVC STL (their C++ standard library) is open-source.
