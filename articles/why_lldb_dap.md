# Why I recommend LLDB-DAP in this tutorial?

Not a strong preference. It seems to work best compared to other debugger extensions (apart from not having good errors on misconfiguration), but use whatever you like more.

Notable alternatives include, in my order of preference:

* [CodeLLDB](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) - Works somewhat well, but was very slow to update for Clang 18, and during those months couldn't be used with LLDB 18 (crashed), so I had to abandon it.

* [Native Debug](https://marketplace.visualstudio.com/items?itemName=webfreak.debug) - This can work with both LLDB and GDB. Works alright, but seems to have issues with displaying variable values. E.g. for `std::vector` it doesn't display the elements in the UI (at least by default).

  Using it with LLDB requires installing LLDB-MI in addition to the LLDB itself (`pacman -S mingw-w64-clang-x86_64-lldb-mi`).

* [C/C++ extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) - In theory supports both LLDB, GDB, and its own debugger (coming from Visual Studio?). In practice, I couldn't get it to work with LLDB (them treating third-party debuggers as second-class citizens is unsurprising).

  Also it has infamously annoying UI, printing a long command to the terminal every time you start debugging, and requiring prepending `-exec` to every command in the debug
