# Why I don't suggest using Visual Studio in this tutorial?

**TL;DR:** VS is easier to set up than VSC, so it often gets recommended to newbies for this reason. But VSC isn't *that* hard to set up either. It requires some minimal knowledge, such as how to use a terminal, and this tutorial covers all of it, without going into too much detail. This has to be learned sooner or later anyway. If a person is unable to do this with a tutorial, they'll going to have a hard time with programming even if they switch to an easier IDE.

---

Visual Studio (not to be confused with Visual Studio Code) is popular, and there's nothing wrong with using it. But I think there are better option, especially not for personal projects, so in this tutorial I recommend Visual Studio Code over it. Why?

## Good things about VS

* **Extremely easy to get started.** This is why it's often recommended to newbies, though we shouldn't make it the only criteria.

* **Great debugger UI.** Probably the best one I've seen so far.

* **Some other tools**, such as a good UI for the profiler.

## Bad things about VS

* **Not cross-platform.** VS locks you to Windows, while VSC can be used on any OS.

* **Not open-source.** If Microsoft makes a change you or the community don't like, there's nothing you can do.

* **Promoting questionable defaults**, some of them hard to change.

  The default build system is MSBuild, and [the default compiler is MSVC](/tooling/articles/choosing_compiler_and_more.md#choosing-a-compiler). It's possible to switch to e.g. Clang and CMake for a nicer experience. But running completely custom build commands will be painful, and so would e.g. trying to update Clang to a newer version that didn't come preinstalled with VS.

  VSC on the other hand, [has no defaults](/tooling/images/bad_defaults_or_no_defaults.jpg) and tries to stay compatible with different tools and compilers. But this does result in higher upfront complexity of knowing what tools to install and how to configure them.
