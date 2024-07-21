# Configuring code completion

## What is code completion?

"Code completion" (or "IntelliSense" as Microsoft calls it in their products) is a feature of most IDEs. It provides suggestions as you type, and shows most errors immediately without having to compile.

[![Clangd intellisense in action, 1](/images/clangd_in_action_1.png)](/images/clangd_in_action_1.png) [![Clangd intellisense in action, 2](/images/clangd_in_action_2.png)](/images/clangd_in_action_2.png)

You don't *have* to use code completion, and writing a few simple programs without code completion is a good learning experience. But for more complex projects it's invaluable.

## Different code completion providers

Out of the box VSC doesn't have code completion for most languages. You must install an extension for it.

The two most popular extensions for C/C++ code completion are:

* [**Microsoft's C/C++ extension**](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools), seemingly using the same code completion engine as Visual Studio.

* [**Clangd.**](https://clangd.llvm.org/) Not to be confused with Clang. Clangd is a code completion engine based on Clang the compiler, hence its name.

We will be installing Clangd, since in my experience it works faster, [doesn't choke on advanced C++ features](TODO show example), and can be used in most popular IDEs (not only in VSC).

## Installing Clangd

Clangd needs two parts to work:

* A extension for your IDE (for VSC in this case).
* A program called `clangd` that the extension interacts with.

Install the latter in MSYS2 using `pacman -S mingw-w64-clang-x86_64-clang-tools-extra`. *([Why not the official Clang installer?](/why_not_official_clang_installer.md))*

In VSC, open the extension marketplace by pressing the 'boxes' button on the left:<br/>
![extensions marketplace icon](/images/vsc_extensions_icon.png)

Search for `clangd` and install it:<br/>
![clangd extension icon](/images/clangd_extension_icon.png)

**NOTE:** You don't need the C/C++ extension to be installed, since Clangd (and others I'll suggest next) replaces it. If you want to keep it installed, you need to disable its intellisense to avoid conflicts:

![clangd disables intellisense](/images/clangd_disables_intellisense.png)

**NOTE:** If Clangd suggests downloading a "language server", refuse. It's not useful since we already installed the same thing from MSYS2, and has [the same downsides as the official Clang installer](/why_not_official_clang_installer.md).

![clangd suggests download](/images/clangd_suggests_download.png)

If you accidentally clicked `Install`, go to `File`->`Preferences`->`Settings`, search for `clangd path` (it will be set to `C:\Users\Username\AppData\...`), click the 'gear' icon to the left of it, then press `Reset`.

If you see this message, you didn't configure PATH correctly as [was explained here](/working_in_vscode_terminal.md). (Alternatively, you can go to `File`->`Preferences`->`Settings`, search for `clangd path`, and put `C:\msys64\clang64\bin\clangd.exe` in there.)

If you change PATH, you'll need to start VSC for it to take effect. And when changing Clangd settings, you can either restart VSC or press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`clangd: Restart language server`.

## Verifying that Clangd works

Create a new file and save it with the `.cpp` extension (e.g. name it `prog.cpp`). You should see `clangd: idle` appear at the bottom-left corner.

When you start typing, you should see red squiggles for invalid code, and code completion suggestions.

## Useful hotkeys

* <kbd>Ctrl</kbd><kbd>Space</kbd> - [Open code completion suggestions.](/images/clangd_in_action_1.png) Normally this happens automatically as you type, but you can open it manually too. Hit <kbd>Escape</kbd> to hide it.

* <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>Space</kbd> - [Open function parameters help.](/images/clangd_in_action_3.png) This also usually opens automatically. Hit <kbd>Escape</kbd> to hide it.

* <kbd>Ctrl</kbd><kbd>.</kbd> - Quick actions. If you see a lightbulb icon, pressing this will suggest to fix the current error, or do something else.

* <kbd>F8</kbd> - [Show next problem.](/images/clangd_in_action_2.png) This explains what a red squiggle means (or you can move your mouse over it).

* Holding <kbd>Ctrl</kbd> and clicking on something in the code will open its declaration or show where it's used.

## Improving Clangd experience

Try to use Clangd for a bit! See what features you like or don't like.

Then come back here and see if you want to change any of the settings below.

For now, I suggest continuing to the next chapter: [configuring hotkeys for compilation](/configuring_vsc_tasks.md).

### Hiding inlay hints

If you type e.g. `std::cout << std::sin(42) << '\n';`, Clangd will show a little gray `x:` before `42`, telling you the function parameter name:

[![inlay hints demo](/images/clangd_inlay_hints.png)](/images/clangd_inlay_hints.png)

This is not a part of the actual source code, just something it displays to make your life easier.

If you find this is annoying, go to `File`->`Preferences`->`Settings`, **search for `inlay hints`, and set it to `offUnlessPressed`**. You can then hold <kbd>Ctrl</kbd><kbd>Alt</kbd> to temporarily unhide them.

### Disabling automatic header insertion

You will see a little dot before some of the Clangd code completions:

[![inlay hints demo](/images/clangd_header_suggestions.png)](/images/clangd_header_suggestions.png)

Accepting such completion by pressing <kbd>Enter</kbd> will automatically add an include for this name, e.g. `#include <iostream>` in this case.

If you don't like this behavior (and arguably it's not good for learning purposes), go to the settings, search `clangd arguments`, press `Add item`, type **`--header-insertion=never`** and press `OK`. Hit <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`clangd: Restart language server` to apply changes.

Even after this, you can still use <kbd>Ctrl</kbd><kbd>.</kbd> (see above) to automatically insert missing headers.

### Disabling function argument placeholders

Sometimes accepting a function name completion will insert placeholders for its arguments. E.g. typing `std::find_` and acceping `std::find_if` will result in this:

[![function argument placeholders demo](/images/clangd_arg_placeholders.png)](/images/clangd_arg_placeholders.png)

If you don't like this and would prefer to get just `std::find_if()`, go to the settings, search `clangd arguments`, press `Add item`, type **`--function-arg-placeholders=false`** and press `OK`. Hit <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`clangd: Restart language server` to apply changes.

### Hiding the `.cache` directory

If you use `File`->`Open Folder...` and look at the `Explorer` tab, you'll see that Clangd has created a `.cache` directory to store its internal data next to your source code.

You can hide this directory from Explorer by going to the settings, searching for `files exclude`, pressing `Add Pattern` and adding `.cache/` to the list.
