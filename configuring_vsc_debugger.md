# How to use a debugger in VSC?

[You should already know how to use a debugger in a terminal.](/debugging_in_terminal.md)

This isn't very convenient, but there's a nicer way of using it in VSC.

## Checking PATH settings

Make sure you can run `lldb --version` in the VSC terminal. If you see `lldb : The term 'lldb' is not recognized...`, make sure you [have LLDB installed](/debugging_in_terminal.md), and [have PATH configured correctly](/working_in_vscode_terminal.md).

## Installing LLDB-DAP

There are several different extensions for debugging C/C++ in VSC:

* [**Microsoft's C/C++ extension**](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

* [**LLDB-DAP**](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.lldb-dap)

We'll be using LLDB-DAP because it works better with MinGW in my experience, and because you can use it in most IDEs, not only in VSC.

Like [Clangd](/configuring_code_completion.md#installing-clangd), LLDB-DAP consists of two parts:

* A plugin for your IDE (for VSC in this case).

  Search for `lldb-dap` in the extension marketplace and install it:<br/>
  [![lldb-dap extension icon](/images/lldb_dap_extension_icon.png)]((/images/lldb_dap_extension_icon.png))

* A program called `lldb-dap` that the extension interacts with. You already have it installed because you [installed LLDB](/debugging_in_terminal.md). But you need to tell the extension where to find it.

  **Open the settings (`File`→`Preferences`→`Settings`), search for `lldb dap executable` and type `C:\msys64\clang64\bin\lldb-dap.exe` in there.**

## Making sure the compilaton settings are correct

Make your compiler flags (in `tasks.json`) include `-g`, [as was explained before](/debugging_in_terminal.md). Make sure to recompile the executable after adding it.

## Configuring the debugger

Make sure you have a folder opened in VSC. Use `File`→`Open Folder...`.

Open the `Run and Debug` tab by pressing the button on the left:<br/>
![run and debug icon](/images/vsc_debugging_icon.png)

Click `create a launch.json file`, and select `LLDB DAP Adapter`. It will create a file called `launch.json` in the `.vscode` directory in the current directory. This file holds debugging configurations. You can create it manually once you get used to it.

[![generated launch.json](/images/generated_launch_json.png)](/images/generated_launch_json.png)

Replace `<your program>` with the name of your program, e.g. `"program": "${workspaceRoot}/prog.exe"`. (Here `${workspaceRoot}` means the currently opened directory.)

Now pressing <kbd>F5</kbd> or the green 'play' button to start the debugger. [Like before](/debugging_in_terminal.md), you should see your application window flash for a moment, and close immediately.

If nothing happens, you likely didn't configure LLDB-DAP [as was explained above](#installing-lldb-dap).

The output of your program should be visible in the `Debug Console` at the bottom of the screen (enable it in `View`→`Debug Console` if it's hidden).

## Using the debugger

The experience is similar to [what you had in the terminal](/debugging_in_terminal.md).

Place a breakpoint by clicking to the left of a line number:

[![placing breakpoint](/images/vsc_breakpoint.png)](/images/vsc_breakpoint.png)

When you start the application and it gets paused on a breakpoint, you'll see the current line in yellow:

[![placing breakpoint](/images/vsc_paused_on_breakpoint.png)](/images/vsc_paused_on_breakpoint.png)

Control the debugger using the panel on the top:

[![debugger controls](/images/vsc_debugger_controls.png)](/images/vsc_debugger_controls.png)

View the variable values on the left panel, or just by moving the mouse over them in the code.

In the `Debug Console`, you can use [any commands you have learned before](/debugging_in_terminal.md), such as `p` to print the values of variables. Or type variable names directly to print them.

Most of the things you see in the UI correspond to the [debugger commands](/debugging_in_terminal.md) you've already learned:

[![debugger layout](/images/vsc_debugger_layout.png)](/images/vsc_debugger_layout.png)

## Automatically rebuilding the program before debugging

Right now, you need to manually compile the program (in the terminal, or with <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>B</kbd>) after every change to the code, before starting the debugger (by pressing <kbd>F5</kbd> or the green arrow).

This can be automated. After the `"cwd": ...` line in `tasks.json`, add `"preLaunchTask": "Compile"` (where `Compile` is the task name that you specified in `tasks.json` as described [here](/configuring_vsc_tasks.md)).

Now pressing <kbd>F5</kbd> or the green arrow will automatically recompile the program before starting the debugger.

## Improving LLDB experience

There are a few extra settings you can add to `launch.json`:

* **Open `Debug Console` automatically when starting the debugger:**
  ```json
  "internalConsoleOptions": "openOnSessionStart",
  ```

* **Improve how complex values are displayed:**
  ```json
  "enableAutoVariableSummaries": true,
  ```
  Let's say you have a struct: (if you don't know what those are, read your C++ book more and come back later)
  ```cpp
  struct A
  {
      int x, y;
  };

  A a = {10, 20};
  ```
  By default it's displayed this way:

  [![no auto variable summary in lldb](/images/lldb_no_auto_var_summaries.png)](/images/lldb_no_auto_var_summaries.png)

  To view the values of `x` and `y`, you'd have to press `>` on the left.

  But with this setting enabled, it will be displayed like this:

  [![auto variable summary in lldb](/images/lldb_auto_var_summaries.png)](/images/lldb_auto_var_summaries.png)

* **Allow inspecting true members of standard classes:**
  ```json
  "enableSyntheticChildDebugging": true
  ```
  Let's say you have a vector: (if you don't know what those are, read your C++ book more and come back later)
  ```cpp
  std::vector<int> v = {1, 2, 3};
  ```
  By default it's displayed like this:

  [![no synthetic child](/images/lldb_no_synth_child.png)](/images/lldb_no_synth_child.png)

  The elements are shown, but the internal implementation details of `std::vector` aren't shown. Sometimes they can be useful.

  With this setting enabled, you'll see this instead:

  [![no synthetic child](/images/lldb_synth_child.png)](/images/lldb_synth_child.png)

  In addition to the elements, this now shows the internals of `std::vector` too.

  By default the `[raw]` section is collapsed, so this isn't too obtrusive.

<!-- TODO when updating to clang 19, check that , works alright in complex situations (test locally a bunch), then add here -->
