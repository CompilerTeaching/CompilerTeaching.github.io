---
title: Compilation model
layout: default
---

The MysoreScript is a simple two-tier implementation of a language.
The first tier is an interpreter, which can be extended to collect profiling information.
The second tier is an LLVM-based JIT.

The basic implementation does not perform on-stack replacement (also known as *deoptimisation* or *side exits*) and transfers control between the interpreter only at function boundaries.

The compiler follows an object oriented style, with each AST node knowing how to interpret and compile itself.
The root class for all AST nodes is [`AST::Statement`](doxygen/struct_a_s_t_1_1_statement.html).
This defines `interpret` and `compile` methods.
Subclasses of [`AST::Expression`](doxygen/class_a_s_t_1_1_expression.html) share an implementation of `interpret` which calls `evaluate`, allowing them to return a value.
This method calls `isConstantExpression` and caches the return value (effectively performing constant folding) and calls the `evaluateExpr`, allowing subclasses to perform the evaluation.
Initially, all functions are executed by the interpreter.

After a [`ClosureDecl`](doxygen/struct_a_s_t_1_1_closure_decl.html) has executed `ClosureDecl::compileThreshold` times, the compiler will run and provide a compiled implementation of the function (in MysoreScript, closures and methods are represented by the same class).

Calling methods and Closures 
----------------------------

The `evaluateExpr` method of the [`AST::Call`](doxygen/struct_a_s_t_1_1_call.html) expression class always performs a normal C function call in the same way, irrespective of whether the target function is compiled or interpreted.
The compiled code works in the same way, so we'll just look at the interpreter for now.
Calling a method is two-stage process:

1. The interpreter calls [`compiledMethodForSelector`](doxygen/namespace_mysore_script.html#aae5e122085fee7efeff56c471a59b304), which returns a pointer to the method.
2. The interpreter calls [`callCompiledMethod`](doxygen/namespace_mysore_script.html#aebfd6603112c6bad1003f055df6cc1b4), which casts the function pointer to one accepting the provided number of arguments and then calls it.

If the method has been compiled, then the function pointer is the one returned by the LLVM JIT.
When calling a method that has not yet been compiled, the returned function pointer will be a trampoline, for example [`methodTrampoline0`](doxygen/namespaceanonymous__namespace_02interpreter_8cc_03.html#ab1c8dadbc4ba7dd91daf20def56ef843) (the zero-argument method trampoline), which looks up the correct AST node and then invokes the interpreter.
This mechanism currently limits the interpreter to handling methods with no more than 10 arguments, a limit that could be improved with a small amount of run-time code generation.
You could try replacing the static trampolines with some simple JIT'd functions generated with LLVM.
