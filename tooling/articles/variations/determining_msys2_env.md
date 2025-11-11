# Determining what you have installed in MSYS2

Ok, so you already have **a** compiler installed in MSYS2. But since it provides multiple different kinds compilers (different versions of GCC and Clang, read [this](/tooling/articles/msys2_environments.md) for more details), we need to determine what exactly you have.

Open MSYS2 terminal (any of them, doesn't matter) and type:
```sh
pacman -Qe
```
It'll list the things ("packages") you have installed in MSYS2.

Depending on what you see, proceed to one of the following links. If there are several matches, you can pick one freely.

* `mingw-w64-clang-x86_64-clang` → This is already what this tutorial recommends, you can follow it as is. [Go to the previous page.](/tooling/articles/installing_toolchain.md)

* `mingw-w64-ucrt-x86_64-gcc` → [GCC in MSYS2 UCRT64](./ucrt64_gcc.md), the default choice for GCC.

* `mingw-w64-x86_64-gcc` → [GCC in MSYS2 MINGW64](./mingw64_gcc.md), GCC with the older C standard library.

* `mingw-w64-ucrt-x86_64-clang` → [Clang in MSYS2 UCRT64](./ucrt64_clang.md) — Clang but in a GCC-centric environment. Makes it easier to switch between GCC and Clang. Lacks the address sanitizer.

* `mingw-w64-x86_64-clang` → [Clang in MSYS2 MINGW64](./mingw64_clang.md) — same, but also with the older C standard library.

* Just `gcc` or `clang` → a really bad idea. Uninstall them using (e.g. `pacman -Rcs gcc` for `gcc`), then proceed with the tutorial normally. See [this](/tooling/articles/choosing_compiler_and_more.md#) for why it is a bad idea.

* None of the above:

  * If the only difference is that you have `-i686-` instead of `-x86_64-`, it means you're using a 32-bit compiler. I recommend stopping living in the past, but if you insist, you can follow the respective link above, but mentally replacing `-x86_64-` with `-i686`, and `...64` with `...32` (e.g. `MINGW32`).

  * If you use `-aarch64` instead of `-x86_64-`, you probably already know what you're doing, and can figure out how to adapt this tutorial for your purposes.

  * If you don't see any of this, then you probably just installed MSYS2, but didn't install a compiler in it. Proceed with the tutorial normally.

The differences between all those options are explained in excruciating detail [here](/tooling/articles/choosing_compiler_and_more.md).

---

[↲ Back to installing the toolchain.](/tooling/articles/installing_toolchain.md)
