# Why do I recommend those specific tools?

This tutorial recommends Clang, MSYS2, Visual Studio Code, Clangd, LLDB-DAP.

This page explains my criteria for picking tools. For explanations on specific tools, see: [Choosing a compiler](/articles/choosing_compiler_and_more.md), [Why MSYS2?](/articles/why_msys2.md), [Why Clangd?](/articles/why_clangd.md), [Why LLDB-DAP?](/articles/why_lldb_dap.md) and others.

## Open source

All tools I recommend here are free and open-source.

This means they will continue to exist as long as there's a big enough community interest in them.

If any questionable decisions are made by the developers, the community (or even you) can make patches or forks.

Them being free makes them accessible to young children and students, so you don't have to use inferior tools for teaching.

Additionally, nobody can take those tools away from you, if your government does something stupid and/or has a political breakdown with the country the developers live in.

## Cross-platform

All tools recommended here are cross-platform (with the exception of MSYS2; though most tools installed *in* it are themselves cross-platform).

If you decide to switch your OS (or your work requires you to), you'll be able to enjoy all the same tools.

## Modular

All tools recommended here are *modular*.

E.g. if you decide VSCode is not for you, nothing locks you into it. Clangd and LLDB-DAP work the same way on most popular IDEs.

## Non-goal: *Excessively* minimizing the entry barrier

I see e.g. Visual Studio being recommended a lot, because it just works and doesn't require any knowledge to configure. Which for some people trumps all criteria above, which I believe isn't a good thing.

It's not like VSC requires any advanced knowledge to configure either, other than knowing what is a terminal and how to compile in the terminal.

## Non-goal: Using "official" tools

Most often I see this with MSVC (or [its ABI](/articles/choosing_compiler_and_more.md)) being recommended over MinGW because Microsoft made them, which supposedly makes them "more native" and inherently better on Windows.

People often struggle to explain what effect this actually has in practice, and why this makes them better.

Though I do agree that a tool being popular and having a community is important, see the [next item](#non-goal-using-the-most-popular-tools).

## Non-goal: Using *the* most popular tools

The most popular tools are not necessarily the best tools.

Using *extremely* unpopular tools would be painful (libraries not supporting obscure compilers and OSes, and so on), so having at least *some* community around a tool is important.

But when choosing between several *somewhat* popular tools, picking the most popular one isn't always the best move.
