---
layout: default
title: Building LLVM
github: llvm-mirror
---

Note for Cambridge students: Debug and release builds of LLVM installed on the filer in `/auto/groups/acs-software/L25/llvm/` and `/auto/groups/acs-software/L25/llvm-release`, respectively.
If you are working on any of the ACS lab machines then you can use these directly.

The examples on this site all depend on LLVM.
The fastest way to check out LLVM is from the [GitHub mirror](https://github.com/llvm-mirror/llvm).
This also allows you to have easily move between releases: checkouts from subversion require either checking out the entire tree including all branches (very large!) or doing a new checkout for each tag.

Dependencies
------------

To build LLVM, you will need a C++11 stack (compiler and standard library).
On FreeBSD 10 or later and Mac OS X, the standard toolchain include a recent clang and libc++, so this will work out of the box.
On Linux systems, this varies a bit, but g++4.8 or later and the associated libstdc++ version will probably work.

You will also need [CMake](http://www.cmake.org) and [Ninja](https://martine.github.io/ninja/).
Ninja is not strictly required, but it makes incremental builds significantly faster and the rest of the instructions will assume that you have it.
On Debian and Fedora systems, the Ninja package is called `ninja-build`, on most other platforms it is simply `ninja`.

Get the code
------------

You probably want to get both LLVM and clang (the [Objective-]{C,C++} front end), as the [SimplePass](SimplePass) example expects that you have clang installed to generate IR to feed to your pass, and the [CellAtom](cellatom) example uses it to compile the runtime to bitcode.

To get both, run the following commands somewhere with a reasonable amount of space (a debug build of LLVM will require around 12GB of disk space):

	$ git clone https://github.com/llvm-mirror/llvm
	$ cd llvm/tools
	$ git clone https://github.com/llvm-mirror/clang
	$ cd ..

You will now be in the `llvm` directory, with clang checked out into `llvm/tools/clang`.
Most of the examples will assume that you are using the 3.7 release, so check out that branch from both:

	$ git checkout --track origin/release_37
	$ cd tools/clang
	$ git checkout --track origin/release_37

Building
--------

Linking LLVM can take a long time and while working on the examples you are likely to compile and link a lot of times.
To ease this pain, we'll do a shared library build.

LLVM has a few build flavours.
Debug builds contain all of the debug info (important if you are debugging!) and contain a lot of assertions to check for correct use of LLVM APIs.
During development, you will want to use a debug build.
Unfortunately, these assertions impose a significant performance penalty and an optimised build without them (the Release profile) is around ten times faster.
You will want to use a release build for any performance evaluation.

We'll build both.
First the debug build.

	$ mkdir Debug
	$ cd Debug
	$ cmake .. -DCMAKE_BUILD_TYPE=Debug -DBUILD_SHARED_LIBS=ON -DLLVM_ENABLE_RTTI=ON -G Ninja
	...
	$ ninja
	$ cd ..

Ninja will take a while to run.
It will run one job per core by default.
If you're short on RAM, then you can restrict it a bit with the `-j` flag, for example `-j2` to only do two build steps in parallel.
If you have 1GB of RAM per core (including hyperthreads) then it should be fine.

When this finishes, do the same for the release build:

	$ mkdir Release
	$ cd Release
	$ cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DLLVM_ENABLE_RTTI=ON -G Ninja
	...
	$ ninja
	$ cd ..

You can use LLVM directly from the build directories (`llvm/Debug` and `llvm/Release`).  Each of these contains a copy of `llvm-config`, which will provide the correct paths for linking.
