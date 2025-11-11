# Configuring code completion

## What is code completion?

"Code completion" (or "IntelliSense" as Microsoft calls it in their products) is a feature of most IDEs. It provides suggestions as you type, and shows most errors immediately without having to compile.

[![Clangd intellisense in action, 1](/tooling/images/clangd_in_action_1.png)](/tooling/images/clangd_in_action_1.png) [![Clangd intellisense in action, 2](/tooling/images/clangd_in_action_2.png)](/tooling/images/clangd_in_action_2.png)

You don't *have* to use code completion, and writing a few simple programs without code completion is a good learning experience. But for more complex projects it's invaluable.

## Different code completion providers

Out of the box VSC doesn't have code completion for most languages. You must install an extension for it.

The two most popular extensions for C/C++ code completion are:

* [**Microsoft's C/C++ extension**](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools), seemingly using the same code completion engine as Visual Studio.

* [**Clangd.**](https://clangd.llvm.org/) Not to be confused with Clang. Clangd is a code completion engine based on Clang the compiler, hence its name.

We will be installing Clangd. *([Why Clangd?](why_clangd.md))*

## Installing Clangd

Clangd needs two parts to work:

* A extension for your IDE (for VSC in this case).
* A program called `clangd` that the extension interacts with.

Install the latter in MSYS2 using `pacman -S mingw-w64-clang-x86_64-clang-tools-extra`. *([Why not the official Clang installer?](/tooling/articles/why_not_official_clang_installer.md))*

(⚠ This installation command assumes you've been following the previous chapters of the tutorial as is. If you instead configured MSYS2 in some other manner, you might need a different version of Clangd, since MSYS2 provides multiple. Consult [this page](./variations/determining_msys2_env.md) for more details, or follow the tutorial from the beginning, reinstalling everything exactly as recommended.)

Install the former in VSC. Open the extension marketplace by pressing the 'boxes' button on the left:<br/>
![extensions marketplace icon](/tooling/images/vsc_extensions_icon.png)

Search for `clangd` and install it:<br/>
![clangd extension icon](/tooling/images/clangd_extension_icon.png)

**NOTE:** You don't need the C/C++ extension to be installed, since Clangd (and others I'll suggest next) replaces it. If you want to keep it installed, you need to disable its intellisense to avoid conflicts:

![clangd disables intellisense](/tooling/images/clangd_disables_intellisense.png)

**NOTE:** If Clangd suggests downloading a "language server", refuse. It's not useful since we already installed the same thing from MSYS2, and has [the same downsides as the official Clang installer](/tooling/articles/why_not_official_clang_installer.md).

![clangd suggests download](/tooling/images/clangd_suggests_download.png)

If you accidentally clicked `Install`, go to `File`→`Preferences`→`Settings`, search for `clangd path` (it will be set to `C:\Users\Username\AppData\...`), click the 'gear' icon to the left of it, then press `Reset Setting`.

If you see this message in the first place, you didn't configure PATH correctly as [was explained here](/tooling/articles/working_in_vscode_terminal.md). (Alternatively, you can go to `File`→`Preferences`→`Settings`, search for `clangd path`, and put `C:\msys64\clang64\bin\clangd.exe` in there.) And if that doesn't help either, you likely forgot to run the `pacman -S ...` command mentioned earlier.

Remember that if you change PATH, you'll need to restart VSC for it to take effect. And after changing Clangd settings, you might need to either restart VSC or press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` for them to take effect.

## Verifying that Clangd works

Create a new file and save it with the `.cpp` extension (e.g. name it `prog.cpp`). You should see `clangd: idle` appear at the bottom-left corner.

When you start typing, you should see red squiggles for invalid code, and code completion suggestions.

If you see no squiggles and no suggestions, it means you didn't install something correctly [as was explained above](#installing-clangd).

## Useful hotkeys

* <kbd>Ctrl</kbd><kbd>Space</kbd> - [Open code completion suggestions.](/tooling/images/clangd_in_action_1.png) Normally this happens automatically as you type, but you can open it manually too. Hit <kbd>Escape</kbd> to hide it.

* <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>Space</kbd> - [Open function parameters help.](/tooling/images/clangd_in_action_3.png) This also usually opens automatically. Hit <kbd>Escape</kbd> to hide it. (If you don't know what "functions" are yet, ignore this for now.)

* <kbd>Ctrl</kbd><kbd>.</kbd> - Quick actions. If you see a lightbulb icon, pressing this will suggest to fix the current error, or do something else.

* <kbd>F8</kbd> - [Show next problem.](/tooling/images/clangd_in_action_2.png) This explains what a red squiggle means (or you can move your mouse over it).

* Holding <kbd>Ctrl</kbd> and clicking on something in the code will open its declaration or show where it's used.

## Improving Clangd experience

Try to use Clangd for a bit! See what features you like or don't like.

Then come back here and see if you want to change any of the settings below.

For now, I suggest continuing to the next chapter: [**Configuring hotkeys for compilation**](/tooling/articles/configuring_vsc_tasks.md).

### Hiding inlay hints

If you type e.g. `std::cout << std::sin(42) << '\n';`, Clangd will show a little gray `x:` before `42`, telling you the function parameter name:

[![inlay hints demo](/tooling/images/clangd_inlay_hints.png)](/tooling/images/clangd_inlay_hints.png)

This is not a part of the actual source code, just something it displays to make your life easier.

If you find this is annoying, go to `File`→`Preferences`→`Settings`, **search for `inlay hints`, and set it to `offUnlessPressed`**. You can then hold <kbd>Ctrl</kbd><kbd>Alt</kbd> to temporarily unhide them.

### Disabling automatic header insertion

You will see a little dot before some of the Clangd code completions:

[![inlay hints demo](/tooling/images/clangd_header_suggestions.png)](/tooling/images/clangd_header_suggestions.png)

Accepting such completion by pressing <kbd>Enter</kbd> will automatically add an include for this name, e.g. `#include <iostream>` in this case.

If you don't like this behavior (and arguably it's not good for learning purposes), go to the settings, search `clangd arguments`, press `Add item`, type **`--header-insertion=never`** and press `OK`. Hit <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` to apply changes.

Even after this, you can still use <kbd>Ctrl</kbd><kbd>.</kbd> (see above) to automatically insert missing headers.

### Disabling function argument placeholders

Sometimes accepting a function name completion will insert placeholders for its arguments. E.g. typing `std::find_` and acceping `std::find_if` will result in something like this:

[![function argument placeholders demo](/tooling/images/clangd_arg_placeholders.png)](/tooling/images/clangd_arg_placeholders.png)

If you don't like this and would prefer to get just `std::find_if()`, go to the settings, search `clangd arguments`, press `Add item`, type **`--function-arg-placeholders=false`** and press `OK`. Hit <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`clangd: Restart language server` to apply changes.

### Disabling snippet suggestions

By default Clangd will suggest you certain common code patterns, or "snippets". E.g. starting to type `typedef` or `#include` will produce something like this:

[![code completion snippets](/tooling/images/clangd_snippet_suggestions.png)](/tooling/images/clangd_snippet_suggestions.png)

If you find them annoying like myself, go to the settings, search for *`snippet suggestions`* and disable them.

### Hiding the `.cache` directory

If you use `File`→`Open Folder...` and look at the `Explorer` tab, you will see that Clangd has created a `.cache` directory to store its internal data next to your source code.

You can hide this directory from Explorer by going to the settings, searching for `files exclude`, pressing `Add Pattern` and adding `.cache/` to the list.

### Fixing the broken sticky scroll

VSC has a feature called Sticky Scroll. One way to see it in action is to have a long function that doesn't fit into one screen. E.g.:

```cpp
int main()
{
    // Add empty lines here to make the function longer than your screen.
}
```

Now if you scroll down a bit, you should see `int main()` still being displayed at the top of your screen, even though you've scrolled past it:

[![sticky scroll in action](/tooling/images/vsc_sticky_scroll.png)](/tooling/images/vsc_sticky_scroll.png)

This is what the Sticky Scroll is supposed to do.

But there is a bug ([1](https://github.com/clangd/clangd/issues/2221), [2](https://github.com/microsoft/vscode-cpptools/issues/13574)) that might cause a lone `{` to be displayed there instead (instead of `int main()`), which isn't useful. This happens if you have the Microsoft's C++ extension installed in addition to Clangd.

To fix this, either uninstall the Microsoft's C++ extension (which isn't useful, as Clangd and other extensions I recommend do all the same things). Or if you want to keep it, press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>→`Preferences: Open User Settings (JSON)`, and add the following fragment into it: (as recommended by one of the links above)
```json
"[cpp]": {
    "editor.stickyScroll.defaultModel": "outlineModel",
},
```
This fixes Sticky Scroll for C++. Repeat the same thing with `[c]` and `[cuda-cpp]` to also fix it for C and Cuda respectively.
