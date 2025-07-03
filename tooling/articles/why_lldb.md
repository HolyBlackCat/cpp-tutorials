# Why I recommend LLDB in this tutorial?

This isn't a strong preference.

It seems to work much faster than GDB on large projects, from my experience.

Also, since this tutorial recommends MSYS2 CLANG64 (because it has ASAN), this locks us to LLDB.

In theory LLDB can handle both MinGW and MSVC programs (unlike GDB), but at the time of writing this the MSVC support was buggy.

Additionally, VSC has a [nice extension to interact with LLDB](/tooling/articles/why_lldb_dap.md).
