---
title: Building the AST
layout: default
---

MysoreScript uses [Pegmatite](../pegmatite) for both parsing and AST construction.
Pegmatite is a parsing expression grammar (PEG) library that is designed to allow easy experimentation with language design.
PEGs use a greedy strategy matching strategy with unlimited backtracking (equivalent to unlimited read-ahead).

The grammar for MysoreScript is defined in [`grammar.h`](doxygen/grammar_8hh.html).
Note that this file just describes what the language looks like: it defines a *recogniser* for the language, not a parser.
This separation makes it easy to reuse a grammar description for syntax highlighting, autocompletion, and so on.
The Pegmatite documentation contains more information about [defining a grammar](../pegmatite/grammar.html)

Pegmatite parsers are created by declaratively associating [AST](doxygen/namespace_a_s_t.html) classes with rules in the grammar.
In MysoreScript, the [`MysoreScriptParser` class in `parser.hh`](doxygen/parser_8hh_source.html) is responsible for reating these links, for example defining that the [`assignment`](doxygen/struct_parser_1_1_mysore_script_grammar.html#aba95df5ac2aad2a4d22a009ba0894388) rule in the grammar is handled by the [`Assignment`](oxygen/struct_a_s_t_1_1_assignment.html) class.

Each AST node constructs itself by popping existing AST nodes off a stack and then pushing itself.
The [`Assignment`](oxygen/struct_a_s_t_1_1_assignment.html) class, for example, declares two fields that use Pegmatite's [`ASTPtr`](../pegmatite/doxygen/classpegmatite_1_1_a_s_t_ptr.html) template.
These register themselves with their container on construction ([`Assignment`](oxygen/struct_a_s_t_1_1_assignment.html) is a subclass of Pegmatite's [`ASTContainer`](pegmatite/doxygen/classpegmatite_1_1_a_s_t_container.html) class) and the container initialises them in reverse order by popping values from the stack.
Pegmatite will therefore construct an [`Assignment`](oxygen/struct_a_s_t_1_1_assignment.html) AST noede by first popping a pointer to an [`Expression`](doxygen/class_a_s_t_1_1_expression.html) object into the [`expr`](doxygen/struct_a_s_t_1_1_assignment.html#a33aaba16dca86a076a2b693eea4b55c2) field and then popping a pointer to a [`VarRef`](doxygen/struct_a_s_t_1_1_var_ref.html) into the [`target`](doxygen/struct_a_s_t_1_1_assignment.html#ac1a45d5c449a138a21d306593d89eb11) field (MysoreScriptParser does not support expressions that evaluate to l-values and so the target of an assignment is always the name of a variable).

Once Pegmatite has constructed an AST, the next step is to perform some semantic analysis.
MysoreScript doesn't do very much of this, because it's intended as a simple system for teaching rather than a production-quality language implementation.  
It does; however, need to create symbol tables to let the interpreter and compiler associate variable references with their declarations and, more importantly, with their locations.

The [`AST::Statement`](doxygen/struct_a_s_t_1_1_statement.html) class defines a [`collectVarUses`](doxygen/struct_a_s_t_1_1_statement.html#a9690633ac88fd1780d7e609b70fbcb35) method, which all concrete subclasses must implement.
This passes two sets, one containing the names of declared variables and one of referenced variables.
This is used when executing a closure, to determine which variables are in each scope.
The first time that a closure is executed, it will invoke this method on each statement, which will recursively find all of the declarations and variable references.
If you are implementing a new AST node that contains others, don't forget to implement this to forward the invocation to subclasses!

**NOTE:** There is no real error checking for variable references, the MysoreScript interpreter will simply crash if you try to access a variable that is not in any symbol tables.
