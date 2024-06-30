# Automating compilation in VSC

Compiling manually in the terminal gets old very quickly. You can automate this in VSC by adding hotkeys and/or buttons in the interface that run the compilation command for you.

We will be using something called "tasks". We won't be using the infamous Code Runner extension. *([Why not Code Runner?](/why_not_code_runner.md))*

## What is a "task" in VSC?

A "task" in VSC is a prepared shell command, that you can run by pressing a hotkey or a button in the UI.

If you don't understand what "shell command" is, go read [Terminal for Dummies](/terminal_for_dummies.md) and [Compiling in the terminal](/compiling_in_terminal_win.md).

## How do I create a task?

First, make sure you have opened a directory, not just a single file. `File`->`Open Folder...`.

Press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`Tasks: Configure Task`->`Create tasks.json from template`->`Others`. This creates a directory called `.vscode` in the current directory, and in it a file called `tasks.json`. (Once you get used to it, you can create this file manually.)

You should see this file:

[![generated tasks.json](/images/generated_tasks_json.png)](/images/generated_tasks_json.png)

This is a task named `echo`, that runs the command `echo Hello` which prints `Hello` (the label and the command don't have to match, you can change the label to anything).

To run this task, press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`Tasks: Run Task`->`echo` (which is your label).

It will ask you `Select for which kind of errors and warnings to scan the task output`, select `Never scan the task output for this task` and observe it adding `"problemMatcher": []` to `tasks.json`. (This refers to displaying red squiggles based on the errors printed by the compiler; you don't need this because Clangd already draws the squiggles for you, while this feature requires complex manual configuration, and doesn't work well in the first place).

I know, I know, this doesn't look very convenient. Bear with me.

Now lets rewrite this task to perform compilation. Replace `echo Hello` with your compiler command (e.g. `clang++ test.cpp -o test.exe`), and `"label": "echo"` with e.g. `"label": "Compile"`. Run this task and observe that it prints errors if the compilation fails, or prints nothing on success.

If it says `clang++ : The term 'clang++' is not recognized as the name of ...`, you need to read [Working in VSC terminal](/working_in_vscode_terminal_win.md) again and configure PATH as it tells you to.

## Dependent tasks

Now create a second task to run the compiled executable. It could look like this:

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile",
            "type": "shell",
            "command": "clang++ test.cpp -o test.exe",
            "problemMatcher": []
        },
        {
            "label": "Run",
            "type": "shell",
            "command": "./test.exe",
            "problemMatcher": []
        }
    ]
}
```

Run the `Compile` task, then the `Run` task, and observe that it runs your executable.

But running two tasks in a row isn't very convenient. We can automate this. Add `"dependsOn": "Compile"` to the second task:

[![task dependencies](/images/vscode_task_dependencies.png)](/images/vscode_task_dependencies.png)

Now running the `Run` task will run `Compile` first, tand then `Run` itself.

## Running tasks conveniently

Pressing <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`Tasks: Run Task` every time to run a task is silly. There is a better way.

Press <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`Tasks: Configure Default Build Task` and select the `Run` task. Observe that `"group": { "kind": "build", "isDefault": true }` got added to your task.

Now pressing <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>B</kbd> will run this task.

## Buttons for tasks

As an alternative to hotkeys, you can create buttons for the tasks.

There are several extensions that can do this. I prefer this one, but you can use any of them:

[![task runner extension icon](/images/task_runner_extension_icon.png)](/images/task_runner_extension_icon.png)

It will add a `Task Runner` panel to the bottom of the `Explorer` tab:

[![task runner panel](/images/task_runner_panel.png)](/images/task_runner_panel.png)

Clicking on a task will run it.

This extension doesn't auto-update the list of tasks, use <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>P</kbd>->`Reload Window` to update it. Or search for a different extension.

## Custom task hotkeys

[The method above](#running-tasks-conveniently) lets you assign a shortcut to only one task. If you want multiple shortcuts, there's a different approach.

Open `File`->`Preferences`->`Keyboard Shortcuts`. Click `Open Keyboard Shortcuts (JSON)` in the top-right corner.

This file holds all your custom keyboard shortcuts. Paste following into it:
```json
// Place your key bindings in this file to override the defaults
[
    {
        "key": "alt+e",
        "command": "workbench.action.tasks.runTask",
        "args": "Run"
    },
    {
        "key": "ctrl+alt+e",
        "command": "workbench.action.tasks.runTask",
        "args": "Compile"
    }
]
```

Now you can run the respective tasks by pressing those shortcuts.