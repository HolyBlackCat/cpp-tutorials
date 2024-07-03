# How to debug issues with DLLs (on MinGW)?

## What is a DLL?

The executables you create are not standalone. When ran, they will immediately load a few so-caleld "DLL"s, and fail to start if they can't find them.

DLLs are files with the `.dll` extension. Some of them were installed with your compiler (look in `C:\msys64\ucrt64\bin`), some are preinstalled with Windows.

DLLs contain functions (compiled functions, not the source code) for your application to execute, and other things. By default your application loads DLLs to get access to some standard library functions.

"DLL" stands for "dynamic link library" ("library" means "a collection of functions (and other things)", and "dynamic linking" refers to loading a library when the program runs, as opposed to baking it into the executable when creating it).

## Why DLLs exist?

Why do some functions have to be in separate files, why can't everything be in a single `.exe`?

1. Several applications can use the same DLLs, saving disk space (DLLs are also called "shared libraries", since they can be *shared* between applications).

2. A user can update the library themselves, without having to recompile the application (to get access to new bugfixes, etc).

## What DLLs my application uses?
