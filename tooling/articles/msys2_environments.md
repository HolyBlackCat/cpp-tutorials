# What are MSYS2 environments?

MSYS2 has something called ["environments"](https://www.msys2.org/docs/environments/): `CLANG64`, `UCRT64`, `MINGW64`, [`MSYS`](#the-msys-environment) and others.

**MSYS2 has multiple versions of each compiler and library, one per environment.** The compilers in each environment [have slighly different settings](#difference-between-the-environments). Some compilers/tools are missing from some environments.

This tutorial uses CLANG64 by default (but [can be adapted](/tooling/articles/installing_toolchain_msys2.md#alternative-choices) to other ones too). UCRT64 is the officially recommended default. MINGW64 is also popular.

* **Each environment has its own packages** (compilers, libraries, tools, etc). The packages for each environment can be distinguished by the **common prefix in their names**, e.g. `mingw-w64-clang-x86_64-...` for CLANG64.

* **Each environment installs its packages to a separate directory** under `C:\msys64`. E.g. CLANG64 installs to `C:\msys64\clang64`.

* **Each environment has its own shortcut in the Start menu**, which opens a terminal with the respective directory added to [PATH](/tooling/articles/terminal_for_dummies.md#what-is-path):

    [![msys2 environment shortcuts](/tooling/images/msys2_env_shortcuts.png)](/tooling/images/msys2_env_shortcuts.png)

    The current environment is displayed in the terminal in magenta text:

    [![terminal displays current msys2 environment](/tooling/images/terminal_shows_msys2_env.png)](/tooling/images/terminal_shows_msys2_env.png)

    E.g. `MSYS2 CLANG64` shortcut starts a terminal with `C:\msys64\clang64\bin` in the PATH.

    All MSYS2 terminals ignore the system-wide PATH setting by default, and set a fully custom PATH for themselves.

## How to use a specific environment?

* **Use the correct shortcut.** (see above)

* **Install the packages with the right prefix.** E.g. in CLANG64 install packages named `mingw-w64-clang-x86_64-...`.

  Run `echo $MINGW_PACKAGE_PREFIX` in MSYS2 terminal to know the right prefix for your environment.

  ⚠ Don't install unprefixed packages for compilers and libraries. Only use them for simple command-line tools (`grep`, `sed`, etc), only if there's no prefixed alternative. ([See below for more details.](#the-msys-environment)) There are exceptions from this rule, e.g. [both `make` and `mingw-w64-...-make` are viable](/tooling/articles/different_flavors_of_make.md).

* **If you want to add MSYS2 to PATH, add the correct directory**: `C:\msys64\clang64\bin` for CLANG64 and so on.

## The `MSYS` environment

One of the environments is called `MSYS`, and it's special.

Don't confuse it with the whole MSYS2, and don't confuse it with the ancient MSYS1 (which sometimes was called just "MSYS").

* It hosts the system packages that don't belong to other environments (such as `pacman`).

* Package names for `MSYS` are **not prefixed** with anything. (`pacman -S clang` installs MSYS Clang). They are installed to `C:\msys64\usr`, and some directly to `C:\msys64`.

* Packages in MSYS [are based on Cygwin](/tooling/articles/why_msys2.md#msys2-and-cygwin) (or rather on MSYS2's own fork of it).

  **MSYS compilers create Cygwin applications**, which [isn't something you normally want](/tooling/articles/why_msys2.md#mingw-vs-cygwin) for your applications. **Don't install MSYS compilers and libraries** unless you know what you're doing. They are also often outdated.

* **All other environments "inherit" from `MSYS`**, in the sense that they add it to the PATH (they add `C:\msys64\usr\bin` after `C:\msys64\<environment>\bin`).

* **⚠ MSYS environment always being in PATH can cause confusion.**

  E.g. if you install `pacman -S gcc` AND don't install `mingw-w64-...-gcc` for your environment, and then try to run `gcc` from your environment's MSYS terminal, you'll end up running MSYS GCC (`C:\msys64\usr\bin\gcc.exe`), which won't be compatible with any libraries you install in your environment, and so on.

## Difference between the environments

MINGW64, UCRT64, and CLANG64 are the three popular environments.

### CLANG64 vs UCRT64/MINGW64

CLANG64 is limited to the Clang compiler and the tools associated with it ([libc++ only, no libstdc++](/tooling/articles/choosing_compiler_and_more.md#choosing-c-standard-library); LLD linker only, no LD; `llvm-ar` only, no Binutils `ar`, etc). While UCRT64 and MINGW64 have **both** Clang and GCC, and both sets of tools.

On the other side, CLANG64 supports [sanitizers](/tooling/articles/choosing_compiler_and_more.md), unlike other environments. And (in rare cases), Clang-based tools seem to work a bit better in it (e.g. at the time of writing, LLDB spams harmless warnings to the console on UCRT64).

UCRT64 is the closest equivalent for CLANG64 that's not limited to Clang/LLVM tools.

In UCRT64 and MINGW64, GCC is considered to be the primary compiler, most packages there are compiled with it (by MSYS2 developers).

#### C++ standard library implementation

In UCRT64 and MINGW64, Clang uses [libstdc++](/tooling/articles/msys2_environments.md) (GCC's C++ standard library) by default (instead of Clang's own [libc++](/tooling/articles/msys2_environments.md), which has to be installed separately with `pacman -S ...-libc++` and enabled by passing `-stdlib=libc++` compiler flag).

#### Linker

In UCRT64 and MINGW64, both GCC and Clang use the LD linker by default, but Clang's LLD linker can be installed separately (`pacman -S ...-lld`) and enabled (in either compiler) using `-fuse-ld=lld` flag (when linking). Since LLD is way faster, it might be a good idea to always use it. Unlike LLD, LD is [sensitive to the order of the `-l` flags](/tooling/articles/using_libraries_pacman.md#determining-compiler-flags-using-pkgconf).

### MINGW64 vs UCRT64/CLANG64

MINGW64 uses the old implementation of the C standard library (Microsoft's `msvcrt.dll`), while UCRT64 and CLANG64 use the new one (`ucrtbase.dll`, also by Microsoft).

`ucrtbase.dll` adds support for UTF-8 in paths (which has to be manually enabled), and otherwise there's not much difference.

### Additional information

Consult [Choosing the compiler](/tooling/articles/choosing_compiler_and_more.md) for more information.

### Rarely used environments

* #### `...32`

  For most `...64` environments, there are `...32` counterparts that compile 32-bit applications as opposed to 64-bit. (`CLANG32` vs `CLANG64`, etc.)

  Those are being gradually phased out, and there's little reason to use them.

* #### `CLANGARM64`

  This can be used to compile ARM applications for Windows ARM laptops. Though they can run normal (x86/x86_64) applications as well via emulation.
