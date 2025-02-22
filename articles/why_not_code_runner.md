# Why do I not recommend using the Code Runner extension?

The Code Runner extension for VSC is quite popular, many beginner tutorials recommend using it. I don't.

It's good for compiling single-file programs with some default compiler settings. When your program consists of multiple source files, or you need to customize compiler settings per project, Code Runner is of no use.

Instead I suggest [using "tasks"](/articles/configuring_vsc_tasks.md), which is a built-in mechanism VSC has for this purpose. It requires a bit more manual configuration, but is a lot more flexibe.
