# Why is Visual Studio Code an IDE?

Newbies are often told that VSCode is "not an IDE", and that they should instead install "an IDE" like VS, CLion, etc.

I don't think this is a useful thing to say.

What people *actually* mean by this is that VSC requires more manual configuration (too much for their taste), and that newbies would be better off using something simpler. This is a valid thing to say, even if [I don't entirely agree with it](/tooling/articles/why_vsc_over_vs.md).

But if this is what you want to say, say so explicitly. Framing it as "IDE vs not IDE" is confusing, because when a newbie only has a vague idea that they "need something called an IDE to code", this gives them an impression that VSC is an entirely unrelated tool (isn't something programmers write code in), and that they *must* pick something else to proceed.

There isn't a meaningful definition of "IDE" that allows VS/Clion/etc, but rejects VSC. It has all the basic features you'd expect from an IDE: syntax highlighting, code completion, debugger UI, symbol browser, Git integration, etc.

If it lacks more advanced features that you consider important, it could be a *bad* IDE for you, but an IDE nontheless.

---

Here are some more common objections that I've heard:

* "It requires plugins to support language X or feature Y" — Ok, but so does VS. Its installer acts as a plugin manager, and you need to manually enable C++ support there.

* "An IDE must include a compiler" feels arbitrary too. Would you stop calling VS an IDE if you had to run a separate installer to get the toolchain? Or if VSC had a plugin that installs the compiler for you, would you start calling it one? (Plugins are fair game, see previous point.) Also see some examples of IDEs not including compilers on [Wikipedia](https://en.wikipedia.org/wiki/Integrated_development_environment).

* "VSC doesn't have feature X" (where X = "profiler UI", for example) also doesn't make much sense, since it's an arbitrary cutoff point. Would VS stop being an IDE if the profiler was removed? An so on.
