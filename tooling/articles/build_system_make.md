# How to use the Make build system?

## About this article

This is a short introduction to Make. If you want to know more, read [the official manual](https://www.gnu.org/software/make/manual/make.html) after finishing this.

Despite being short, I try to teach how to write actually **decent** makefiles. Sadly there are many bad tutorials out there.

## Installation

Run **`pacman -S make`** to install `make`. [Note that we're not installing `mingw-...-make`](/tooling/articles/different_flavors_of_make.md), though it's a viable option too.

If you're going to use Make outside of the MSYS2 terminal, it's a good idea to add it to the PATH (read [Terminal for Dummies](/tooling/articles/terminal_for_dummies.md) first if you don't know what that is). Since it's installed to a different directory than your compiler, it needs to be added separately. Add `C:\msys64\usr\bin` to the same place where you added `C:\msys64\clang64\bin`, **RIGHT AFTER** `C:\msys64\clang64\bin`. It needs to be after, or [bad things](/tooling/articles/msys2_environments.md#the-msys-environment) will happen. Don't forget to restart VSC and/or your terminal after changing the PATH to make them see the updated value.

Check that it was installed correctly by running `make --version`.

## Makefiles

A configuration for Make is called a "makefile". Create a file named `Makefile` in your project directory.

VSC might suggest you to install an extension to deal with makefiles. I recommend not using it for now to avoid confusion, you can always try it out later if you want.

Let's start with the following simple makefile:

```make
prog.exe:
	clang++ prog.cpp -o prog.exe
```
(Note that the second line starts with a Tab character. Make is sensitive to this, and won't accept a few spaces in its place.)

A makefile is a list of rules ("recipes"), telling Make how create certain files ("targets"). This one says that to create `prog.exe` Make needs to run `clang++ prog.cpp -o prog.exe`.

Now if you run `make` in your project directory, it should run this command to compile the file.

**BUT!** If `prog.exe` already exists, it will instead say `make: 'prog.exe' is up to date.` and not do anything. This isn't good, because if you change `prog.cpp` now, the executable won't be rebuilt.

## Prerequisites

Let's now try this makefile:
```make
prog.exe: prog.cpp
	clang++ prog.cpp -o prog.exe
```

This says that `prog.cpp` is a **prerequisite** for `prog.exe`.

If you run `make` now, it will rebuild `prog.exe` only if the **modification time** of `prog.cpp` is newer than that of `prog.exe`. The OS tracks when every file was last modified. When you edit `prog.cpp` in your text editor and save it, its modification time will be updated. When you then run `make`, it will see that the `.cpp` file was modified since the last compilation, and will rebuild your `.exe`. Since now the `.exe` has a newer modification time, the subsequent runs of `make` will not do anything.

This means that Make will only compile the `.cpp` files that have changed since the last build.

This is already very useful, and you can modify your [`tasks.json`](/tooling/articles/configuring_vsc_tasks.md) to run `make` instead of `clang++ ...`. This way clicking `Run` will not recompile your file every time if it's not needed.

## Multiple source files

You could adapt the makefile above for multiple source files:

```make
prog.exe: 1.cpp 2.cpp
	clang++ 1.cpp 2.cpp -o prog.exe
```

Note the multiple prerequisites. This means `prog.exe` will be rebuilt if **any of** the prerequisites is newer than `prog.exe` (if any source file was changed since the last compilation).

This isn't the best approach [for the reasons already explained](/tooling/articles/multifile_programs.md#the-simplest-approach).

## Multiple recipes

Now lets use the makefile to perform the proper [multi-step build procedure](/tooling/articles/multifile_programs.md#the-proper-procedure) you learned before.

```make
1.o: 1.cpp
	clang++ 1.cpp -c -o 1.o

2.o: 2.cpp
	clang++ 2.cpp -c -o 2.o

prog.exe: 1.o 2.o
	clang++ 1.o 2.o -o prog
```

Notice the prerequisites for the linking being the `.o` files.

Setting correct prerequisites is important not only because Make compares modification times, but also because it ensures the correct **build order** (so that Make doesn't do the linking before the compilation, before the results of compilation are specified as prerequisites for linking).

Now if you try to run `make`, it should build your executable, right?... Wrong. You'll notice it only builds `1.o`. By default Make builds the first target in the makefile. You can either put `prog.exe: ...` first, or run `make prog.exe` to tell it what it needs to build, or set the `.DEFAULT_GOAL := prog.exe` variable somewhere to override the default target.

Let's try again:
```make
1.o: 1.cpp
	clang++ 1.cpp -c -o 1.o

2.o: 2.cpp
	clang++ 2.cpp -c -o 2.o

.DEFAULT_GOAL := prog.exe
prog.exe: 1.o 2.o
	clang++ 1.o 2.o -o prog.exe
```
This should now build `prog.exe` correctly.

So we're all good now? Not yet. The big issue is that when you **modify a header (`2.h`)**, nothing is rebuilt because Make knows nothing about it. We'll get to it in a moment, but for now...

## Phony targets

Make has so-called "phony" targets ("phony" means ["not genuine or real"](https://www.merriam-webster.com/dictionary/phony)). Unlike normal targets they don't produce a file.

They can be used to run arbitrary commands, e.g. this one cleans the compilation results.

```make
.PHONY: clean
clean:
	rm -f *.o *.exe
```

You can run this target using `make clean`.

Bad tutorials often omit the `.PHONY: clean` part (which marks the target as phony), but if you've been paying attention, you should be able to guess why this is problematic. If you create a file named `clean`, then without this mark, this target wouldn't run.

## Parallel builds

By default Make builds one file at a time. Pass `-j...` flag to build in parallel, where `...` is the number of processes to run in parallel (usually should match the number of the CPU cores you have). E.g. `-j4` builds using 4 parallel jobs.

One thing you can do is `-j$(nproc)`. This automatically uses the number of cores you have. `$(...)` runs the program you tell it, `nproc` in this case (which prints the number of cores), and substitutes whatever it prints to the command. This is not a feature of Make, but of your shell.

**WARNING:** Make has a major footgun. Passing `-j` without a number will build using an infinite number of jobs (all commands will run at the same time). If you try this on a large project, this will exhaust all your RAM and can hang your PC. Don't do that. Another fun thing is that some [shells](/tooling/articles/terminal_for_dummies.md#what-is-a-shell) (Bash, I'm looking at you), if `nproc` is not installed, silently ignore the error and remove `$(nproc)`, leading to this issue.

This can be automated, [more on that below](#automatic-parallel-builds).

## Simplifying the makefile

### Automatic Make variables

The makefile we have right now is way too verbose. We can do better. First we can change the commands to use Make's "automatic" (built-in) variables:
```make
1.o: 1.cpp
	clang++ $< -c -o $@

2.o: 2.cpp
	clang++ $< -c -o $@

.DEFAULT_GOAL := prog.exe
prog.exe: 1.o 2.o
	clang++ $^ -o $@

.PHONY: clean
clean:
	rm -f *.o *.exe
```

This file does the exact same thing as before. Here `$...` are Make's built-in variables. `$@` refers to the current target file, `$^` refers to all prerequisites, and `$<` refers to the first prerequisite.

Astute readers might be wondering why we aren't using `$^` instead of `$<`, since there's only one prerequisite anyway. And indeed it doesn't matter in this example. It will matter in the next chapter though, we'll get to that.

### Pattern rules

We can do even better. Make has something called "pattern rules":

```make
%.o: %.cpp
	clang++ $< -c -o $@

.DEFAULT_GOAL := prog.exe
prog.exe: 1.o 2.o
	clang++ $^ -o $@

.PHONY: clean
clean:
	rm -f *.o *.exe
```
Since all `.o` files are created using the same procedure anyway, we can combine their recipes into one `%.o: %.cpp` "pattern" recipte.

### Variables

Lastly we can move the list of the source files to a variable. While we're at it, let's make variables for the compiler and flags too.
```make
objects := 1.o 2.o

CXX := clang++
CXXFLAGS := -g
LDFLAGS :=

%.o: %.cpp
	$(CXX) $< -c -o $@ $(CXXFLAGS)

.DEFAULT_GOAL := prog.exe
prog.exe: $(objects)
	$(CXX) $^ -o $@ $(CXXFLAGS) $(LDFLAGS)

.PHONY: clean
clean:
	rm -f *.o *.exe
```
Those variable names [follow a specific convention](https://www.gnu.org/software/make/manual/make.html#Makefile-Conventions-1) that Make users expect. The same convention expects `CXXFLAGS` to be used even when linking. (If you're programming in C rather than C++, use `CC` for the compiler and `CFLAGS` for the compiler flags.)

Listing object files instead of source files is a bit ugly. Let's change that to the source files:
```make
sources := 1.cpp 2.cpp

CXX := clang++
CXXFLAGS := -g
LDFLAGS :=

%.o: %.cpp
	$(CXX) $< -c -o $@ $(CXXFLAGS)

.DEFAULT_GOAL := prog.exe
prog.exe: $(sources:.cpp=.o)
	$(CXX) $^ -o $@ $(CXXFLAGS) $(LDFLAGS)

.PHONY: clean
clean:
	rm -f *.o *.exe
```
Here `:.cpp=.o` replaces the `.cpp` suffix in each list element with `.o`.

## Detecting header changes

Now finally to address the elephant in the room.

So far our makefile wasn't handling header changes correctly, as was mentioned before. Changing our `2.h` doesn't cause the source files to be rebuilt, but it should.

You could add `2.h` manually as a prerequisite, but this gets really annoying really fast when you have many headers. We can automate this.

First, add `-MMD -MP` to the compiler flags. (Here I'm adding them outside of the variable, to prevent them from being accidentally removed, but it doesn't really matter.)

```make
sources := 1.cpp 2.cpp

CXX := clang++
CXXFLAGS := -g
LDFLAGS :=

%.o: %.cpp
	$(CXX) $< -c -o $@ -MMD -MP $(CXXFLAGS)

.DEFAULT_GOAL := prog.exe
prog.exe: $(sources:.cpp=.o)
	$(CXX) $^ -o $@ $(CXXFLAGS) $(LDFLAGS)

.PHONY: clean
clean:
	rm -f *.o *.d *.exe
```

Now if you recompile the source files, you should see files `1.d`,`2.d` being generated. (Notice that we also added them to `clean`.)

If you open one of those files, you should see something like this:

```make
1.o: 1.cpp 2.h
2.h:
```
Do you see where this is going? Those are the lists of prerequisites we need, automatically generated by the compiler.

Now we just need to hook them up to the makefile, using `include`:

```make
sources := 1.cpp 2.cpp

CXX := clang++
CXXFLAGS := -g
LDFLAGS :=

%.o: %.cpp
	$(CXX) $< -c -o $@ -MMD -MP $(CXXFLAGS)

.DEFAULT_GOAL := prog.exe
prog.exe: $(sources:.cpp=.o)
	$(CXX) $^ -o $@ $(CXXFLAGS) $(LDFLAGS)

.PHONY: clean
clean:
	rm -f *.o *.d *.exe

-include $(sources:.cpp=.d)
```

`-` before the `include` suppresses the error if the files don't exist, which is a good thing, because they won't exist during the initial compilation.

This is it, our final makefile. There are more improvements that could be made here, but this is the minimal sane version that is actually usable in practice.

All the other build systems like CMake work similarly (rely on the `.d` files). More advanced build systems often convert `.d` to some other more compact format to speed up compilation (reading tons of `.d`s on every compilation seems to be the main reason why Make is slow on large projects).

There are some variations of this trick. For example, `-MD -MP` (instead of `-MMD -MP`) will also track the system headers, which is slower, but rebuilds correctly if you update the system libraries (or third-party ones). Some other variations run the compiler twice: one normally and one to generate the `.d` file, but this looks inferior to what we're doing here.

# Advanced makefile usage

The above is the minimal makefile knowledge that you should have. You can stop here and explore the other build systems.

But if you decide you want to continue using makefiles as your build system, here are some tricks you might find useful.

## Built-in rules

Make has some built-in rules. For example, if you remove the `%.o: %.cpp` rule, it will do something similar automatically just because the `.o` files are mentioned as the prerequisites of `prog.exe`.

This can often get confusing, so if you're not relying on those rules, you should disable them. Either by passing the `-R` flag to `make` every time, or by adding `MAKEFLAGS += -R` somewhere near the beginning of your makefile.

## Automatic parallel builds

[Passing `-j...`](#parallel-builds) can be automated. You can do `MAKEFLAGS += -j...` inside of the makefile.

Here is how you do it properly:
```make
JOBS := $(shell nproc)
ifneq ($(JOBS),)
MAKEFLAGS += -j$(JOBS)
endif
```
Firstly `$(shell nproc)` in a makefile is equivalent to `$(nproc)` in shell. It runs the command and returns whatever it has printed.

Since it will silently return nothing if `nproc` is not installed, we must protect against that, to avoid [the footgun](#parallel-builds) mentioned before. `ifneq ($(JOBS),)` compares `JOBS` with an empty string, and only applies it if non-empty.

## Moving the built files to a directory

It's often desirable to have the `.exe`, `.o`, and `.d` in a subdirectory, e.g. to be able to easily delete the entire thing at once, or just to avoid it cluttering the view.

Buckle up:

```make
MAKEFLAGS += -R

JOBS := $(shell nproc)
ifneq ($(JOBS),)
MAKEFLAGS += -j$(JOBS)
endif

sources := 1.cpp 2.cpp

CXX := clang++
CXXFLAGS := -g
LDFLAGS :=

build_dir := build
bin_dir := $(build_dir)/bin
obj_dir := $(build_dir)/obj

$(bin_dir) $(obj_dir):
	mkdir -p $@

$(obj_dir)/%.o: %.cpp | $(obj_dir)
	$(CXX) $< -c -o $@ -MMD -MP $(CXXFLAGS)

.DEFAULT_GOAL := $(bin_dir)/prog.exe
$(bin_dir)/prog.exe: $(sources:%.cpp=$(obj_dir)/%.o) | $(bin_dir)
	$(CXX) $^ -o $@ $(CXXFLAGS) $(LDFLAGS)

.PHONY: clean
clean:
	rm -rf $(build_dir)

-include $(sources:%.cpp=$(obj_dir)/%.d)
```
First we create variables `build_dir`, `bin_dir`, `obj_dir` to hold the names of our directories.

We also define a rule to create the directories (`$(bin_dir) $(obj_dir):`). Notice that you can have more than one target in a rule, this is a shorthand for writing **multiple** individual rules.

We add `| $(obj_dir)` prerequisite to the object files, to automatically create this directory before building them.

The `|` separates ["order-only prerequisites"](https://www.gnu.org/software/make/manual/make.html#Types-of-Prerequisites). The short explanation is that this prevents unnecessary rebuilds when modification time of the **directory itself** changes (which happens when you add or remove contents to it). You should use `|` any time your prerequisite is a directory that needs to be created.

We use a slightly different form of `$(a:x=y)` here, with the `%` signs. This is necessary because we now not only want to alter the suffix, but also add a prefix (`1.cpp` -> `build/obj/1.o`).

And also now `clean:` can just remove the build directory, instead of the individual files.

And lastly, one clever thing you can do here is replace `$(bin_dir) $(obj_dir):` with `%/:` to automatically create any directory mentioned as a prerequisite. But then you must make sure those prerequisites are spelled with `/` at the end.

## Overriding variables

Any variable in the makefile can be overridden with a flag. E.g. `make JOBS=42`. In this specific case it's not very useful, since `make -j...` overrides the `-j...` in `MAKEFLAGS`, but it's often useful in other cases.

Note that `JOBS=42 make` doesn't work (this is an environment variable now). It only works if you do `JOBS=42 make -e`, or if the makefile doesn't define the `JOBS` variable at all (the environment variable will automatically get picked up as a Make variable), or if the makefile sets it using `JOBS ?= whatever` (this form accepts environment variables too).

In any case, if the makefile defines the variable using `override JOBS := whatever`, it will not accept any user ever.

I recommend marking all variables `override` by default. if you don't want them to be overridden. The conventional variables like `CXX`, `CXXFLAGS`, `LDFLAGS` should be overridable, so don't mark them.

## Automatically finding the source files

It can be desirable to automatically find all source files, instead of listing them manually in the makefile.

This is easy, replace `1.cpp 2.cpp` with `$(wildcard *.cpp)`. I also recommend moving the source files into some directory and using `$(wildcard source/*.cpp)`. This not only looks cleaner, but the wildcard will also work faster if it has less files to check.

Some people dislike this approach, because it makes the builds slower, because we need to rescan the directories every time. It should be fine for small projects though.

### Finding the source files recursively

Note that `wildcard` doesn't look into subdirectories. If your files are spread across multiple subdirectories, you need another approach.

Here's a function that searches recursively:
```make
# $1 is a list of directories, $2 is a list of patterns.
override rwildcard = $(foreach d,$(wildcard $(1:=/*)),$(call rwildcard,$d,$2) $(filter $(subst *,%,$2),$d))
```
You can add it to your makefile, and then do the following, assuming you created a `source` directory for your source files.
```make
sources := $(call rwilrdcard,source,*.cpp)
```
(This can search in `.` too if you pass that, but that is unwise, as it will have to search though a lot of junk.)

#### Directory prerequisites for files in multiple directories

Setting up directory prerequisites for the source files spread across multiple directories can be a bit tricky. The best way I've found is as follows.

Ideally, we'd want something like this: `$(obj_dir)/%.o: %.cpp | $(dir $@)`. (But this doesn't work, see below.)

Here `$(dir )` is a built-in function that rermoves the last component from the path, so `$(dir a/b/c.cpp)` is `a/b/`. This would work really well with the `%/:` (universal target for creating directories) explain above.

But sadly this doesn't work directly, because `$@` doesn't work in the prerequisite list. There is a way to make it work though.

Add [`.SECONDEXPANSION:`](https://www.gnu.org/software/make/manual/html_node/Secondary-Expansion.html) somewhere near the top of the makefile (on a separate line), and then use:
```make
$(obj_dir)/%.o: %.cpp | $$(dir $$@)
```
`SECONDEXPANSION` is a fairly obscure feature. It makes Make scan (expand variables and functions) in the prerequisite lists *twice*. During the second expansion `$@` is accessible, so we can use it. Using `$$` instead of `$` escapes the `$` to delay it until the second expansion, because we don't want it to be expanded to early (because during the first expansion `$@` returns an empty string).


### Cross-platform makefiles

#### Operating systems

Supporting other OSes in a makefile usually requires only minimal changes. In our case the only change needed is removing the `.exe` extension if we're not on Windows.

First of all, we need to detect if we're on Windows or not. Here's one way to do it:

```make
ifeq ($(OS),Windows_NT)
TARGET_OS := windows
else
TARGET_OS := other
endif
```
And then:
```make
ifeq ($(TARGET_OS),windows)
EXT_EXE := .exe
else
EXT_EXE :=
endif
```
And use `prog$(EXT_EXE)` instead of `prog.exe`.

The reason why we use two separate `if`s is to allow [overriding](#overriding-variables) `TARGET_OS`, and then respecting the override when selecting the extension.

There's another detection mechanism: running [`uname`](https://linux.die.net/man/1/uname) and examining the OS name it prints. For example, this detects Linux:
```make
ifeq ($(shell uname -o),GNU/Linux)
```
This isn't 100% reliable on Windows, because if the user is running [`mingw32-make` instead of `make`](./different_flavors_of_make.md) and didn't add `uname` to the PATH, it won't work. And running `uname` is probably slower too. So my recommendation would be to check the `$(OS)` environment variable first, and if we're not Windows (and you need a more detailed detection, e.g. Linux vs Mac), *then* run `uname`.

#### Compilers

GCC and Clang mostly have compatible compiler flags, so you don't need to do much here.

MSVC not only has different flags, but also can't generate [the `.d` files](#detecting-header-changes), so if you need to support it, it's easier to use a different build system (though it's technically possible to generate the `.d` files yourself, by using `/showIncludes` MSVC flag and parsing its output, that's not trivial).
