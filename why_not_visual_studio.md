# Why I don't suggest using Visual Studio in this tutorial?

Visual Studio (not to be confused with Visual Studio Code) is immensely popular.

There's nothing wrong with using it, but I don't consider it to be the best choice, and in this tutorial I recommend Visual Studio Code over it. Why?

## Good things about VS

* **Extremely easy to get started.** This is why it's often recommended to newbies, though we shouldn't make it the only criteria.

* **Great UI for the debugger.** Probably the best one I've seen so far.

* **Other convenient tools**, such as a good UI for the profiler.

## Bad things about VS

* **Not cross-platform.** VS locks you to Windows, while VSC can be used on any OS.

* **Not open-source.** If Microsoft makes a change you or the community don't like, there's nothing you can do.

* **Promoting questionable defaults**, some of them hard to change.

  The default build system is MSBuild, and [the default compiler is MSVC](/choosing_compiler_and_more.md#choosing-a-compiler). It's possible to switch to e.g. Clang and CMake for a nicer experience. (But running completely custom build commands will be painful, or e.g. trying to update Clang to a newer version that didn't come preinstalled with VS.)

  VSC on the other hand, [has no defaults](/images/bad_defaults_or_no_defaults.jpg) and tries to stay compatible with different tools and compilers. But this does result in higher upfront complexity of knowing what tools to install and how to configure them.
