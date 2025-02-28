# How to use the Make build system?

## About this article

This is a short introduction to Make. If you want to know more, read [the official manual](https://www.gnu.org/software/make/manual/make.html) after finishing this.

## Installation

Run **`pacman -S make`** to install `make`. [Note that we're not installing `mingw-...-make`](/articles/different_flavors_of_make.md), though it's a viable option too.

If you're going to use Make outside of the MSYS2 terminal, it's a good idea to add it to the PATH (read [Terminal for Dummies](/articles/terminal_for_dummies.md) first if you don't know what that is). Since it's installed to a different directory than your compiler, it needs to be added separately. Add `C:\msys64\usr\bin` to the same place where you added `C:\msys64\clang64\bin`, **RIGHT AFTER** `C:\msys64\clang64\bin`. It needs to be after, or [bad things](/articles/msys2_environments.md#the-msys-environment) will happen. Don't forget to restart VSC and/or your terminal after changing the PATH to make them see the updated value.

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

This is already very useful, and you can modify your [`tasks.json`](/articles/configuring_vsc_tasks.md) to run `make` instead of `clang++ ...`. This way clicking `Run` will not recompile your file every time if it's not needed.

## Multiple source files

You could adapt the makefile above for multiple source files:

```make
prog.exe: 1.cpp 2.cpp
	clang++ 1.cpp 2.cpp -o prog.exe
```

Note the multiple prerequisites. This means `prog.exe` will be rebuilt if **any of** the prerequisites is newer than `prog.exe` (if any source file was changed since the last compilation).

This isn't the best approach [for the reasons already explained](/articles/multifile_programs.md#the-simplest-approach).

## Multiple recipes

Now lets use the makefile to perform the proper [multi-step build procedure](/articles/multifile_programs.md#the-proper-procedure) you learned before.

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
