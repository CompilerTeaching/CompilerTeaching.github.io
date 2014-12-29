---
title: Debugging grammars
layout: default
---

Debugging parsers can be time consuming and tedious.  Pegmatite contains some
facilities to make life easier in this regard.

{% include contents.markdown %}

Debugging Parsers
-----------------

By default, all of the debugging features are compiled out.  Add
`-DDEBUG_PARSING` to your compiler flags (or simply define the `DEBUG_PARSING`
macro before including the parser header).

You can then use the `trace()` function to generate rules in your grammar.  The
calculator example contains this rule in the grammar:

	Rule mul_op = mul >> '*' >> val;

To inspect when it is matched, we'll modify it to trace attempts to match this
rule:

	Rule mul_op = trace("multiply", mul >> '*' >> val);

Rebuilding and running a couple of examples will show the tracing in the
output.  We'll start with one that shouldn't expect to match the `mul_op` rule:

	1+2
	[0] Trying multiply (line 1, column 1)
	[1]  Trying multiply (line 1, column 1)
	[1]  multiply failed (line 1, column 1)
	[0] multiply failed (line 1, column 2)
	[0] Trying multiply (line 1, column 3)
	[1]  Trying multiply (line 1, column 3)
	[1]  multiply failed (line 1, column 3)
	[0] multiply failed (line 1, column 4)
	success

This will first try to match an add operation, because that has the highest
precedence.  The left hand side of the add is some other expression.  Again add
has the highest precedence, but now is a left recursive rule, so the
backtracking will try a subtraction, and so on until it tries a multiply.  The
multiply will fail after parsing a number, because the next token is not `'*'`.
The failing token is shown in the output (line 1, column 2, the `+`).

The same process occurs on the right-hand side of the add, again failing, this
time on the end-of-input character.  If we try an example that should succeed,
then we'll end up with a simpler trace:

	...
	1*2
	[0] Trying multiply (line 1, column 1)
	[1]  Trying multiply (line 1, column 1)
	[1]  multiply failed (line 1, column 1)
	[0] multiply succeeded (line 1, column 4)

Here, the multiply matches the entire input.  

Debugging AST Building
----------------------

Pegmatite builds ASTs by invoking actions that pop constructed AST nodes off a
stack and push new ones on.  If you define the `DEBUG_AST_CONSTRUCTION` macro,
then the code that constructs the AST will print various tracing information.
Once again, visiting the calculator example we can see this:

	1+2*3
	[0] Constructing AST::Number (0x7fa5d86000f0) off the AST stack
	[0] Constructed AST::Number (0x7fa5d86000f0) off the AST stack
	[1] Constructing AST::Number (0x7fa5d86011b0) off the AST stack
	[1] Constructed AST::Number (0x7fa5d86011b0) off the AST stack
	[2] Constructing AST::Number (0x7fa5d8601170) off the AST stack
	[2] Constructed AST::Number (0x7fa5d8601170) off the AST stack
	[3] Constructing AST::MultiplyExpression (0x7fa5d86011e0) off the AST stack
	[2] Popped AST::Number (0x7fa5d8601170) off the AST stack
	[1] Popped AST::Number (0x7fa5d86011b0) off the AST stack
	[1] Constructed AST::MultiplyExpression (0x7fa5d86011e0) off the AST stack
	[2] Constructing AST::AddExpression (0x7fa5d8601350) off the AST stack
	[1] Popped AST::MultiplyExpression (0x7fa5d86011e0) off the AST stack
	[0] Popped AST::Number (0x7fa5d86000f0) off the AST stack
	[0] Constructed AST::AddExpression (0x7fa5d8601350) off the AST stack

The number in square brackets indicates the number of elements on the stack.
The first six lines construct `AST::Number` instances and push them on the
stack.  The address of the objects is also shown, so you can inspect them in a
debugger.  These are the terminals and so have nothing in between the
'Constructing' and 'Constructed' lines.

The next node to be constructed is the multiply expression (`2*3`).  This pops
the top two numbers off the stack and then pushes itself.  Finally, the
top-level add AST node starts to construct itself, popping the multiply and the
number from the stack and finally pushing itself.

At the end of AST construction, there should be a single root node on the
stack, which is then returned.  Here we can see that this is exactly what
happens: the `AddExpression` finishes constructing itself with 0 nodes on the
stack.

By inspecting this trace, you can usually see where things are being pushed
onto the stack and not popped, or objects are attempting to pop values from the
stack that don't exist.
