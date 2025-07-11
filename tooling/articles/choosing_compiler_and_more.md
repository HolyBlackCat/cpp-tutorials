# Choosing a compiler and more

This is a complicated topic prone to holy wars. Here I try to explain all the choices you have, so you can make an educated decision.

But yltimately, any remotely popular compiler is going to work fine, and the differences will be minor, especially for a newbie.

## What choices you can make?

### Available compilers

The three most popular compilers are: (in no particular order)

* **MSVC** (stands for "**M**icro**s**oft **V**isual **C**++"), the compiler bundled with Visual Studio.
* **GCC**, the compiler commonly used on Linux.

  On Windows it's usually bundled with something called [**MinGW**](https://www.mingw-w64.org/) (a collection of tools and libraries used to compile Windows applications without Visual Studio). So people tend to say "MinGW" to mean "GCC on Windows", and in general don't understand the [difference between the two](./why_msys2.md).

* **Clang**, the most recent addition to the list ("recent" as in "first release was in this millenium").

  On Windows can use MinGW, but doesn't have to.

The pros and cons of each one are explained below.

But wait, there's more. Yeach compiler has its own implementation of the **C++ standard library**:
* **MSVC STL** from MSVC.
* **libstdc++** from GCC.
* **libc++** from Clang.

Some compilers can be used with standard libraries from **other** compilers. This is a fairly common practice, and is in no way worse than using their native libraries. (Which combinations are valid is shown below.)

So not only do you have to choose the compiler, but also separately the C++ standard library. The pros and cons of each one are explained below as well.

And lastly, when using Clang+libc++ on Windows, it can be used in two modes, MinGW-compatible and MSVC-compatible.

In the end, there seems to be 7 valid combinations:

[![tool combinations](../images/tool_combinations.svg)](../images/tool_combinations.svg)

Each colored line is one combination. Red ones are operating in MSVC-compatible mode, blue ones are operating in MinGW-compatible mode. (That is on Windows. Elsewhere there's no MSVC nor MSVC STL, so only the blue ones are avaialble.)

Now the same thing in the form of a table:

&nbsp;|[Compiler](#choosing-a-compiler)|[ABI and various standard libraries](#mingw-abi-vs-msvc-abi)|[C++ standard library](#choosing-c-standard-library)|Available sanitizers|How to install
---|---|---|---|---|---
1|MSVC|MSVC|MSVC STL|ASAN|Comes with [Visual Studio](https://visualstudio.microsoft.com), or use the standalone ["build tools"](https://visualstudio.microsoft.com/downloads).
2|Clang|MSVC|MSVC STL|ASAN**|[Official Clang installer](https://github.com/llvm/llvm-project/releases) (look for `LLVM-...-win64.exe`), also must install MSVC.<br/><sup>(Or you could make MSYS2 Clang operate in MSVC-compatible mode with some command-line flags, but that's extra work.)</sup>
3|Clang|MSVC|Clang's libc++|ASAN, UBSAN|Same, but also [must compile libc++ manually.](https://libcxx.llvm.org/BuildingLibcxx.html)
4|Clang|MinGW|Clang's libc++|ASAN, UBSAN|[MSYS2 CLANG64](/tooling/articles/msys2_environments.md) (other [MSYS2 environments](/tooling/articles/msys2_environments.md) have libc++ too, but without sanitizers).<br/><sup>(Also there are [alternative distributions](/tooling/articles/why_msys2.md).)</sup>
5|Clang|MinGW|GCC's libstdc++|None*|With [MSYS2 UCRT64 or MINGW64](/tooling/articles/msys2_environments.md).<br/><sup>(Could also use official Clang installer with any MinGW, but that requires some custom command-line flags. Also there are [alternative distributions](/tooling/articles/why_msys2.md).)</sup>
6|GCC|MinGW|GCC's libstdc++|None*|With [MSYS2 UCRT64 or MINGW64](/tooling/articles/msys2_environments.md).<br/><sup>(Also there are [alternative distributions](/tooling/articles/why_msys2.md).)</sup>
7|GCC|MinGW|Clang's libc++|None* ?|A rather obscure combination. Same as above, but needs [some custom GCC flags](https://releases.llvm.org/19.1.0/projects/libcxx/docs/UsingLibcxx.html#using-a-custom-built-libc).

You can use any IDE with any of those combinations, with more or less effort (e.g. using Visual Studio while compiling for MinGW might cause [intellisense](https://learn.microsoft.com/en-us/visualstudio/ide/using-intellisense?view=vs-2022) to misbehave).

**"ABI"** stands for ["application binary interface"](https://en.wikipedia.org/wiki/Application_binary_interface). Loosely speaking, if two compilers use the same ABI, you can use any library compiled with one compiler with another. Some C (not C++) libraries might be compatible regardless. (In the explanation above I called those "MinGW-compatible mode" vs "MSVC-compatible mode".)

**"Sanitizers"** are automatic tools that you can enable to help you catch bugs: ASAN ("Address Sanitizer") for pointer errors, UBSAN for some other undefined behavior. (Note that on Linux, both GCC and Clang support both ASAN and UBSAN, plus some sanitizers not available on Windows.)

<sup>* - You can use UBSAN, but [in a weird mode](https://stackoverflow.com/a/59083808/2752075) that doesn't display errors and just crashes when a violation is detected.</sup><br/>
<sup>** - Should work, but didn't immediately work out of the box for me.</sup>

## Choosing a compiler

All three are viable, but here's my personal order or preference:

1. Clang - It's nice to have support for different C++ standard libraries and ABIs. For some platforms it's the only compiler (e.g. Emscripten, Android NDK).

   Most code analysis tools (including those in most IDEs, excluding VS) are based on Clang today, so you'll likely have to keep your code compatible with it regardless.

2. GCC - Ok, but slower compilation speed compared to Clang on Windows, and worse memory usage (about 2x more in my tests). Cross-compilation* requires slightly more effort**. No sanitizers on Windows.
3. MSVC - [Has some issues with optimization quality and standard conformance.](#msvc-issues) Slow at adding support for C++23 and C++26. Not open-source. Cross-compilation is impractical***. But compilation speed is good (same as Clang or a bit faster). Not open-source.

   Has Address Sanitizer on Windows, which is better than GCC (which has no sanitizers on Windows) but worse than Clang (which also has UB Sanitizer on Windows).

<sup>* - Cross-compilation is compiling for one operating system while using another. Such as compiling for Windows while using Linux.</sup><br/>
<sup>** - While the same Clang executable can compile for any platform (if you provide it with the right libraries), GCC needs to be recompiled for each target platform, which takes time.</sup><br/>
<sup>*** - While you can [run MSVC in Wine](https://github.com/mstorsjo/msvc-wine), it's quite slow for most practical purposes.</sup>

## Choosing C++ standard library

There's no single "C++ standard library". Rather, there are several implementations of the same specification:

* Clang's **libc++**

  Some minor features are missing as of libc++-18, such as parallel standard algorithms and iterator validation (the latter is [partially implemented](https://libcxx.llvm.org/Hardening.html#hardened-containers-status), and the address sanitizer partially makes up for it).

  They seem to have made some clever design choices, such as a very memory-efficient SSO for `std::string`s.

  Is the only choice on some platforms (Emscripten, Android NDK).

* **MSVC STL**

  Has some warts that they don't fix to preserve ABI compatibility, such as: [`std::deque` being a joke](https://github.com/microsoft/STL/issues/147), or [a const-correctness bugs in the iterators](https://quuxplusone.github.io/blog/2021/07/14/an-iterator-is-not-a-const-iterator/), or `std::async` non-conformingly using a thread pool, which messes with `thread_local` variables.

  Otherwise seems to be quite decent.

* GCC's **libstdc++**

  The golden standard. Most Linux distributions use it by default.

If you're using GCC or MSVC, you're locked into their respective C++ standard libraries, but Clang can use any of the three.

libstdc++ has iterator validation ([`-D_GLIBCXX_DEBUG`](/tooling/articles/recommended_compiler_flags.md#flags-to-catch-errors)), but on MinGW it doesn't work with the sanitizers (sanitizers only work in [CLANG64 environment](/tooling/articles/msys2_environments.md), which only has libc++ and not libstdc++).

All three big implementations are good. On Linux I personally would prefer libstdc++ for iterator validation, but this tutorial uses libc++ on Windows to [allow sanitizers](#available-compilers).

## MinGW ABI vs MSVC ABI

MinGW ABI and MSVC ABI are incompatible. Libraries compiled with one can't be used with another (except perhaps for some C (not C++) libraries).

If you're using GCC or MSVC, you have no choice.

Clang be switched between MSVC mode and MinGW mode:

* If you're using either libstdc++ or MSVC STL, that locks you into the respective mode.

* If you have prebuilt libraries compiled for one of those two ABIs, and you can't compile them for another (e.g. because they are closed-source), that locks you into that ABI.

* MSVC ABI has bugs/quirks, such as:

  * `[[no_unique_address]]` not working out of the box (must use `[[msvc::no_unique_address]]`).

  * Empty base class optimization [not working in some cases](https://stackoverflow.com/q/12701469/2752075).

  * MSVC, in their C standard library headers, single-handedly "deprecating" some of the standard functions in favor of their non-cross-platform alternatives (e.g. `scanf` vs `scanf_s`; must define `_CRT_SECURE_NO_WARNINGS` to silence this). Strictly saying this isn't an "ABI" issue, but this is something you'll face when using MSVC or Clang in MSVC mode.

MinGW is less popular/has less inertia behind it, which means in rare cases some libraries or applications may not support it. Sometimes you can patch the libraries, but sometimes not. Some libraries do the opposite and only support MinGW.

I have a weak preference towards MinGW ABI (because of the quirks mentioned above), but both are ok.

## MSVC issues

MSVC in general seems to be a rather wonky compiler. Still very popular, presumbly because it's a default choice in VS, but wonky.

Elaborating on the [claims](#choosing-a-compiler) I made above:

* "Issues with standard conformance" - They are getting better at this, but still have some sloppy behavior, such as:

  * `[[no_unique_address]]` having no effect (have to use `[[msvc::no_unique_address]]`).
  * Accepting `typename` in wrong locations (`typename int x;`).
  * [Accepting incorrect `requires`-clauses.](https://stackoverflow.com/q/78155107/2752075)
  * ...

* Being slow at implemeting C++23, and not starting C++26 at all. If you take a look at [cppreference](https://en.cppreference.com/w/cpp/compiler_support/23), at the time of writing, they implement less than half of C++23. Note that I'm talking only about the core language features, not standard library features.

  Historically, they were slow to adopt C++11, then slowly started to overtake GCC and Clang up to C++20, and now they are behind again.

* Tends to have particularly quirky bugs: [[1]](https://stackoverflow.com/q/78799496/2752075), [[2]](https://stackoverflow.com/q/69431575/2752075), [[3]](https://stackoverflow.com/q/78157781/2752075), [[4]](https://gcc.godbolt.org/#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,selection:(endColumn:2,endLineNumber:11,positionColumn:2,positionLineNumber:11,selectionStartColumn:2,selectionStartLineNumber:11,startColumn:2,startLineNumber:11),source:'int+main()%0A%7B%0A++++constexpr+bool+a+%3D+false%3B%0A%0A++++auto+lambda+%3D+%5B%5D%0A++++%7B%0A++++label:%0A++++++++%5B%5Bmaybe_unused%5D%5D+constexpr+bool+a+%3D+true%3B%0A++++++++static_assert(a)%3B+//+Fails+with+%60/std:c%2B%2Blatest%60.+Stops+failing+if+you+either+remove+the+label+or+remove+the+attribute.%0A++++%7D%3B%0A%7D'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:50,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:vcpp_v19_40_VS17_10_x64,filters:(b:'0',binary:'1',binaryObject:'1',commentOnly:'0',debugCalls:'1',demangle:'0',directives:'0',execute:'0',intel:'0',libraryCode:'0',trim:'1',verboseDemangling:'0'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,libs:!(),options:'/std:c%2B%2Blatest',overrides:!(),selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1),l:'5',n:'0',o:'+x64+msvc+v19.40+VS17.10+(Editor+%231)',t:'0')),header:(),k:50,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(compilerName:'x86-64+clang+19.1.0',editorid:1,fontScale:14,fontUsePx:'0',j:1,wrap:'1'),l:'5',n:'0',o:'Output+of+x64+msvc+v19.40+VS17.10+(Compiler+%231)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4), [[5]](https://gcc.godbolt.org/#g:!((g:!((g:!((h:codeEditor,i:(filename:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,selection:(endColumn:2,endLineNumber:11,positionColumn:2,positionLineNumber:11,selectionStartColumn:2,selectionStartLineNumber:11,startColumn:2,startLineNumber:11),source:'template+%3Cbool+nothrow%3E%0Astruct+S+%7B%0A++++//+error+C2065:+!'nothrow!':+undeclared+identifier%0A++++using+type+%3D+int(void*)+noexcept(nothrow)%3B%0A++++type*+member%3B+//+%3D+nullptr%3B+//+Uncommenting+this+removes+the+error.%0A%7D%3B%0A%0Avoid+f()+%7B%0A++++S%3Cfalse%3E+s%3B%0A++++auto+p+%3D+s.member%3B%0A%7D'),l:'5',n:'0',o:'C%2B%2B+source+%231',t:'0')),k:49.80089890958678,l:'4',n:'0',o:'',s:0,t:'0'),(g:!((g:!((h:compiler,i:(compiler:vcpp_v19_latest_x64,filters:(b:'0',binary:'1',binaryObject:'1',commentOnly:'0',debugCalls:'1',demangle:'0',directives:'0',execute:'1',intel:'0',libraryCode:'1',trim:'1',verboseDemangling:'0'),flagsViewOpen:'1',fontScale:14,fontUsePx:'0',j:1,lang:c%2B%2B,libs:!(),options:'',overrides:!(),paneName:'',selection:(endColumn:1,endLineNumber:1,positionColumn:1,positionLineNumber:1,selectionStartColumn:1,selectionStartLineNumber:1,startColumn:1,startLineNumber:1),source:1),l:'5',n:'0',o:'',t:'0')),header:(),k:50.597303271239696,l:'4',m:50,n:'0',o:'',s:0,t:'0'),(g:!((h:output,i:(compilerName:'x86-64+clang+20.1.0',editorid:1,fontScale:14,fontUsePx:'0',j:1,wrap:'1'),l:'5',n:'0',o:'Output+of+x64+msvc+v19.latest+(Compiler+%231)',t:'0')),header:(),l:'4',m:50,n:'0',o:'',s:0,t:'0')),k:50.19910109041324,l:'3',n:'0',o:'',t:'0')),l:'2',n:'0',o:'',t:'0')),version:4).

  Also the buggy empty base optimization, [as mentioned before](https://stackoverflow.com/q/12701469/2752075).

* "Issues with optimization quality" - This is from personal experience, but I keep hearing the same sentiments on the internet: [[1]](https://www.reddit.com/r/cpp/comments/100vctp/msvc_vs_clang_performance_has_anyone_tested/), [[2]](https://www.agner.org/optimize/optimizing_cpp.pdf#page=10), [[3]](https://lemire.me/blog/2023/02/27/visual-studio-versus-clangcl/), etc.

* Not cross-platform:

  * Can only compile for Windows.
  * Can only be used on Windows (not counting weird setups using Wine), while GCC and Clang can cross-compile from other operating systems.

* Not free and open-source:

  Most of time it's just a matter of ideology, but it means you can't fix bugs in it yourself.

  Also there are some weird license requirements, such as apparently not being allowed to [redistribute compiled debug binaries](https://learn.microsoft.com/en-us/cpp/windows/determining-which-dlls-to-redistribute?view=msvc-170) (*"You also can't redistribute debug versions of your applications ..."*).

  Note that MSVC STL (their C++ standard library) is open-source.
