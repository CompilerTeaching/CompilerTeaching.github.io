---
title: SimplePass
layout: default
---

This is a trivial LLVM pass, which can be extended to analyse or transform LLVM
IR.  To build:

	$ mkdir Build
	$ cd Build
	$ cmake ..
	$ make

This will produce a library, called SimplePass.dyld (on OS X) or SimplePass.so
(on other UNIX-like systems).  This library can then be loaded by clang and
will automatically insert itself at the end of the optimisation pipeline.

To use it, run this command from the build directory:

	$ clang -Xclang -load -Xclang ./SimplePass.so -c {some source file} -O1

The pass will not be invoked at -O0, but will at any other optimisation level.
