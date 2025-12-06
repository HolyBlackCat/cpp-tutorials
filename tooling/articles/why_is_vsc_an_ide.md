# Why is Visual Studio Code an IDE?

<sup>(Rant ahead.)</sup>

In some circles, the meaning of the word "IDE" seems to have drifted over time.

Its original (and correct) meaning is "a text editor with certain programming-related features", but now some people understand it as "a text editor with *many* programming-related features, that is sufficiently easy to configure, etc", or something along those lines.

This is problematic for at least two reasons:

* Since it's so vague, no one has come up with coherent critera of what does or doesn't count as an IDE. (See some [failed attempts](#some-failed-attempts-at-classifying-what-does-or-doesnt-count-as-an-ide) below.)

* And more importantly, this is confusing newbies.

Newbies are often told that VSCode is "not an IDE", and that they should instead install "an IDE" like VS, CLion, etc.

I don't think this is a useful thing to say.

What people *actually* mean by this is that VSC requires more manual configuration (too much for their taste), and that newbies would be better off using something simpler. This is a valid thing to say, even if I [don't entirely agree with it](/tooling/articles/why_vsc_over_vs.md).

But if this is what you want to say, say so explicitly. Framing it as "IDE vs not IDE" is confusing, because when a newbie only has a vague idea that they "need something called an IDE to code", this gives them an impression that VSC is an entirely unrelated tool (isn't something programmers write code in), and that they *must* pick something else to proceed.

All the basic features expected from an IDE are there: syntax highlighting, code completion, debugger UI, symbol browser, Git integration, etc.

If it lacks more advanced features that you consider important, it could be a *bad* IDE for you, but an IDE nontheless.

---

### Some failed attempts at classifying what does or doesn't count as an IDE

Here are some more common ones that I've heard.

* "Must not need plugins to support a programming language" — Even VS fails this test, because it has no C++ support out of the box unless enabled in the installer, and that's no different from installing a plugin.

* "Must include a compiler" — See e.g. [Wikipedia](https://en.wikipedia.org/wiki/Integrated_development_environment) for examples of IDEs that don't come with compilers. You probably wouldn't stop calling VS an IDE if it moved the compiler to a separate installer. And if VSC got a plugin that installs the compiler for you (plugins are fair game, see above), would that affect whether you call it an IDE?

* "Must include a specific advanced feature" (like a profiler UI, for example) — This is an arbitrary cutoff point. Would you stop consider VS an IDE if the profiler UI was removed? Etc.
