# Why do I recommend Visual Studio Code rather than Visual Studio?

VSC is open-source and cross-platform, it can be used with any compilers, build systems, etc.

VS has some nice features, like a good debugger UI, profiler UI, etc., but it locks you to Windows, to MSVC (or Clang in MSVC-compatible mode), makes it difficult to use all but certain build systems (MSBuild and CMake are supported directly, everything else needs to generate VS project files), etc.

VS is easier to set up than VSC, so it often gets recommended to newbies for this reason. But VSC isn't *that* hard to set up either. It requires some minimal knowledge, such as how to use a terminal, and this tutorial covers all of it, without going into excessive detail. This has to be learned sooner or later anyway. If a person is incapable of running a few terminal commands that a tutorial tells them to, they'll going to have a hard time with programming even if they switch to an easier IDE.
