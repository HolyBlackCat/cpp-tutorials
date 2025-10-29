# Why do I recommend those specific tools?

This tutorial recommends Clang, MSYS2, Visual Studio Code, Clangd, LLDB and LLDB-DAP, as I consider them to be the best choice.

This page explains my criteria for picking tools. For explanations on specific tools and choices, see:<br/>
[Why MinGW?](/tooling/articles/why_mingw.md) (also covers different C++ standard libraries)<br/>
[Choosing a compiler](/tooling/articles/choosing_compiler_and_more.md) (also covers different C++ standard libraries)<br/>
[Why MSYS2?](/tooling/articles/why_msys2.md) (also explains in detail what is MinGW)<br/>
[Why Clangd?](/tooling/articles/why_clangd.md)<br/>
[Why LLDB?](/tooling/articles/why_lldb.md)<br/>
[Why LLDB-DAP?](/tooling/articles/why_lldb_dap.md)<br/>
[Why not Visual Studio?](/tooling/articles/why_not_visual_studio.md)<br/>
[Why not the official Clang installer?](/tooling/articles/why_not_official_clang_installer.md)<br/>
[Why not WSL?](/tooling/articles/why_not_wsl.md)<br/>

## Open source

All tools I recommend here are free and open-source.

This means they will continue to exist as long as there's a big enough community interest in them.

If any questionable decisions are made by the developers, the community (or even you) can make patches or forks.

Them being free makes them accessible to young children and students, so you don't have to use inferior tools for teaching, nor resort to piracy.

Additionally, nobody can take those tools away from you, if your government does something stupid and/or has a political breakdown with the country the developers live in.

## Cross-platform

All tools recommended here are cross-platform (with the exception of MSYS2; though most tools installed *in* it are themselves cross-platform, so it's not needed if you're not using Windows).

If you decide to switch your OS (or your work requires you to), you'll be able to enjoy all the same familiar tools.

## Modular

All tools recommended here are *modular*.

E.g. if you decide VSCode is not for you, nothing locks you into it. Clangd and LLDB-DAP work the same way on most popular IDEs.

Similarly, VSCode can work with any compiler, debugger, and any build system.

## Non-goal: *Excessively* minimizing the entry barrier

I see e.g. Visual Studio being recommended a lot, because it just works and doesn't require any knowledge to configure. Which for some people trumps all criteria above, which I believe isn't a good thing.

It's not like VSC requires any advanced knowledge to configure, other than knowing what is a terminal and how to compile in the terminal (which is all explained in this tutorial).

## Non-goal: Using "official" tools

Most often I see this with MSVC (or [its ABI](/tooling/articles/choosing_compiler_and_more.md)) being recommended over MinGW because Microsoft made them, which supposedly makes them "more native" and inherently better on Windows.

People often struggle to explain what effect this actually has in practice, and why this makes them better.

Though I do agree that a tool being popular and having a community is important, see the [next item](#non-goal-using-the-most-popular-tools).

## Non-goal: Using *the* most popular tools

The most popular tools are not necessarily the best tools.

Using *extremely* unpopular tools would be painful (libraries not supporting obscure compilers and OSes, and so on), so having at least *some* community around a tool is important.

But when choosing between several *somewhat* popular tools, picking the most popular one isn't always the best move.
