# How to install libraries in MSYS2's `pacman`?

## Check if the library exists in MSYS2's `pacman`

Use `pacman -Ss ...` to search through MSYS2 "packages" (things you can install). Try **`pacman -Ss openal`** to search for OpenAL. It'll print the list of matching packages.

**⚠ NOTE:** As before, we're looking only for things named `mingw-w64-clang-x86_64-...` ([explanation](/tooling/articles/msys2_environments.md)). If you scroll through the list, you'll quickly find this:
```
clang64/mingw-w64-clang-x86_64-openal 1.23.1-2
    OpenAL audio library for use with opengl (mingw-w64)
```
If you can't find your library, it means MSYS2 maintainers didn't add it, and you have to [compile it yourself](/tooling/articles/using_libraries_compiling_manually.md).

## Install the library in MSYS2

Install the library using the name you found above. In our case:

```sh
pacman -S mingw-w64-clang-x86_64-openal
```

Again, make sure the libraries you install [are prefixed with `mingw-w64-clang-x86_64-...`](/tooling/articles/msys2_environments.md).

## Look at what you have installed

If you installed from MSYS2, use `pacman -Ql ...` to list the installed files, e.g. **`pacman -Ql mingw-w64-clang-x86_64-openal`** in our case.

You should see a list of files:

[![openal package contents](/tooling/images/pacman_package_contents.png)](/tooling/images/pacman_package_contents.png)

Here `/clang64/...` refers to the `C:\msys64\clang64\` directory, so you can examine it manually too.

As you can see, you get **headers** (`.h` files), the **compiled library** (`.dll`, `.a`, `.dll.a`), the **`.pc` file** (see below), and some other things.

We're particularly interested in the `.pc` file. It describes the compiler flags needed to use this library. If it's missing, you have to guess the flags.

## Figure out the required compiler flags

Try to compile a small test program using this library. (Find one for your library in its manual.)

Here's a small program using OpenAL that does nothing:
```cpp
#include <alc.h>

int main()
{
    alcOpenDevice(nullptr);
}
```
If you just try to compile it using `clang++ prog.cpp`, you'll get errors. (Try it.) You need to add some compiler flags.

### Determining compiler flags using `pkg-config`

**IF** the `.pc` file exists, we'll use it to figure out the correct compiler flags. For libraries that don't provide it, [see below](#guessing-the-compiler-flags).

First, you need to install `pkg-config`, which is the program that reads those files:

```sh
pacman -S mingw-w64-clang-x86_64-pkgconf
```
(There's also `mingw-w64-clang-x86_64-pkg-config`. Both install a program called `pkg-config`, but `...-pkgconf` is a more modern rewrite of the same thing.)

Run it to get the flags: `pkg-config --libs --cflags openal`.

**NOTE:** The library name (`openal` here) matches the name of the `.pc` file (`openal.pc`). You can also use `pkg-config --list-all` to list all libraries known to `pkg-config` (all installed `.pc` files).

(If you [compile and link separately](/tooling/articles/multifile_programs.md), you can use `pkg-config --cflags openal` and `pkg-config --libs openal` to get compiler flags and linker flags separately.)

You should see output like this:
```sh
-IC:/msys64/clang64/include/AL -lopenal
```

Now try using those flags by compiling the test program above.
```sh
clang++ prog.cpp -IC:/msys64/clang64/include/AL -lopenal
```
If everything is done correctly, this it compile without any errors.

**⚠ NOTE:** You might see red squiggles in VSCode, this isn't a problem. Configuring VSC to understand the library [is a separate issue](TODO_vsc_libs). For now just compile in the terminal, I'll explain this later.

**NOTE:** While most compiler-flags are order-independent, in some cases it might be important to have `-l...` to the right of any `.c`/`.cpp` files. (If you're using [LD linker](/tooling/articles/msys2_environments.md#linker).)

If you're curious, here's a full program that plays a short beep using OpenAL. Try running it. For now, you don't have to understand it

<details><summary>Code</summary>

```cpp
#include <al.h>
#include <alc.h>

#include <chrono>
#include <iostream>
#include <thread>

int main()
{
    ALCdevice *device = alcOpenDevice(nullptr);
    if (!device) {std::cout << "Can't open device!\n"; return 1;}

    ALCcontext *context = alcCreateContext(device, nullptr);
    if (!context) {std::cout << "Can't create context!\n"; return 1;}
    if (!alcMakeContextCurrent(context)) {std::cout << "Can't create make the context current!\n"; return 1;}

    ALuint buffer;
    alGenBuffers(1, &buffer);
    if (!buffer) {std::cout << "Can't create a buffer!\n"; return 1;}

    short sine[24000];
    for (int i = 0; i < 24000; i++)
        sine[i] = short(std::sin(i / 50.) * 32767);
    alBufferData(buffer, AL_FORMAT_MONO16, sine, sizeof sine, 48000);

    ALuint source;
    alGenSources(1, &source);
    if (!source) {std::cout << "Can't create a source!\n"; return 1;}
    alSourcei(source, AL_BUFFER, buffer);

    alSourcePlay(source);

    while (true)
    {
        int state;
        alGetSourcei(source, AL_SOURCE_STATE, &state);
        if (state != AL_PLAYING)
            break;
        std::cout << ".";
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
    std::cout << "\nBye!\n";
}
```

</details>

### Guessing the compiler flags

If a library doesn't have a `.pc` file, or if you want to practice, you can *guess* the correct compiler flags yourself.

### Step 1: Make sure `#include` works

Try including your library. Start with a simple program like this one:
```cpp
#include <alc.h>

int main() {}
```
(Replace the header name with the one for your library. [Look at what headers it installs](#look-at-what-you-have-installed).)

If you compile this without any flags (`clang++ prog.cpp`), you **can** get an error:
```
prog.cpp:1:10: fatal error: 'alc.h' file not found
    1 | #include <alc.h>
      |          ^~~~~~~
1 error generated.
```
In some libraries it will just out of the box, then you can immediately go to the next step.

For OpenAL it doesn't work, because the compiler looks for headers in `C:\msys64\clang64\include`, while `alc.h` is installed into <code>C:\msys64\clang64\include\\<b>AL</b></code> ([as you can see above](#look-at-what-you-have-installed)).

At this point you have two options. You either:

1. Tell the compiler to look in this directory as well: **`-IC:/msys64/clang64/include/AL`**,

    **OR**

2. You add the directory name to `#include`: **`#include <AL/alc.h>`**

For some libraries both approaches work. Some are picky. In any case, read the manual to know how the library prefers you to spell the header ("`<alc.h>`" in our case, or `pkgconf` wouldn't have printed `-I...`), spell it that way, and add `-I` as needed to make it work.

Try compiling the test program again, now with `-I...`, and it should compile successfully:
```cpp
#include <alc.h>

int main() {}
```
```
clang++ prog.cpp -IC:/msys64/clang64/include/AL
```

### Step 2: Make sure calling functions works

The previous step alone isn't enough. If you now try to call any function from this library, you'll see a different error:
```cpp
#include <alc.h>

int main()
{
    alcOpenDevice(nullptr);
}
```
```sh
clang++ prog.cpp -IC:/msys64/clang64/include/AL
```
```
ld.lld: error: undefined symbol: __declspec(dllimport) alcOpenDevice
>>> referenced by C:/msys64/tmp/1-f8cdc7.o:(main)
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
```

If you don't get any errors, the library is likely [header-only](/tooling/articles/using_libraries.md#what-is-a-library), and you can skip this step.

This error happens because the function definitions are in `.a`/`.dll` files ([see above](#look-at-what-you-have-installed)), and you need to point your compiler to them (or [your linker](TODO_multifile_prorams), rather).

You need to use either the `.dll` (dynamic linking) **OR** `.a` (static linking). If you link dynamically, your executable will need a copy of the `.dll` to run (for more information, read [What is a DLL?](/tooling/articles/debugging_dll_issues.md#what-is-a-dll), [Why do DLLs exist?](/tooling/articles/debugging_dll_issues.md#why-do-dlls-exist), and [Static linking](/tooling/articles/debugging_dll_issues.md#static-linking)).

Use one of the two options:

1. **Link dynamically**

   This will use the `lib__.dll.a` file. If you only have `.a` and not `.dll.a`, use option (2) instead.

   Given `lib__.dll.a`, pass the `__` part of its name to the `-l...` flag. (E.g. given `libopenal.dll.a`, use **`-lopenal`**.)

   `.dll.a` is an "import library". It's a tiny file that doesn't contain the function definitions, but will silence the `undefined symbol` errors and cause your application to load `.dll` when started, which *does* contain the definitions.

   Since all `.dll.a`/`.a` files are installed to the same directory (`C:\msys64\clang64\lib`), you normally don't need to manually specify their location. But `-L...` lets you do that if needed (like `-I...` for headers).

2. **Link statically**

   This will use the `lib__.a` file. Use this option if this library has no `.dll.a`, or if you [intentionally want to link statically](/tooling/articles/debugging_dll_issues.md#static-linking).

   Given `lib__.a`, pass the `__` part of its name to the `-l...` flag. (E.g. given `libopenal.a`, use **`-lopenal`**.)

   This embeds the library into the executable (so it can run without the `.dll`, but your users also can't update the library themselves).

   In most libraries you'll notice that `.dll.a` and `.a` have the same name, so how does `-l` select which one to use? (How does `-lopenal` pick between `libopenal.dll.a` vs `libopenal.a`?) It prefers `.dll.a` by default if it exists. Passing [`-static`](/tooling/articles/debugging_dll_issues.md#static-linking) makes it prefer `.a` for all libraries. (It's possible to configure per library, but I'm not going to explain it here.)

Adding the `-l...` flag should fix the error. Run the resulting application to confirm everything works.
