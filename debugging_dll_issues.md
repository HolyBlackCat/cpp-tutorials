# How to debug DLL issues (on MinGW)?

## What is a DLL?

The executables you create are not standalone. When ran, they will immediately load a few so-caleld "DLL"s, and **fail to start if they can't**.

DLLs are files with the `.dll` extension. Some of them were installed with your compiler (look in `C:\msys64\ucrt64\bin`), some are preinstalled with Windows.

DLLs contain functions (compiled functions, not the source code) for your application to execute, and other things. By default your application loads DLLs to get access to some standard library functions.

"DLL" stands for a "dynamic link library" ("library" means "a collection of functions (and other things)", and "dynamic linking" refers to loading a library when the program runs, as opposed to baking it into the executable when creating it).

## Why do DLLs exist?

Why do some functions have to be in separate files, why can't everything be in a single `.exe`?

1. Several applications can use the same DLLs, saving disk space and possibly RAM (DLLs are also called "shared libraries", since they can be *shared* between applications).

2. A user can update the library themselves, without having to recompile the application (to apply bugfixes, etc).

## How does a DLL loading failure look like?

* **`The code execution cannot proceed because __.dll was not found.`** - DLL straight up wasn't found.

* **`procedure entry point __ could not be located in the dynamic link library __`** - a DLL with the correct name was found, but the contents are incompatible. Note that the DLL mentioned in the message isn't necessary the offending one, it can be a red herring.

* **Application simply doesn't start** - this will happen when launching it in a terminal or in an IDE. Double-click it in the explorer and you should see one of the above error messages.

## What DLLs my application uses?
