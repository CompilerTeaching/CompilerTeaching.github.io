---
title: SimplePass
layout: default
github: CompilerTeaching/SimplePass
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

	$ clang -Xclang -load -Xclang ./SimplePass.so -c {some source file}

The pass will be invoked at any optimisation level.
If you are modifying SimplePass then look at the `RegisterStandardPasses` declarations at the bottom of the file to see how it is registered with the pass pipeline.
You may wish to insert your pass at a different point.
