---
layout: default
title: Pegmatite
github: CompilerTeaching/Pegmatite
---

Pegmatite design overview
-------------------------

This is a fork and extensive rewrite of Achilleas Margaritis's ParserLib.  It
has the following goals:

- Idiomatic C++11
- Simple use
- Reuseable, reentrant grammars with multiple action delegates
- No dependency on RTTI / exceptions (usable in embedded contexts)

It has the following explicit non-goals:

- High performance (ease of use or modification should not be sacrificed in the
  name of performance)
- Compatibility with the old ParserLib

Design outline
--------------

All rules should be immutable after creation.  Ideally they'd be constexpr, but
this is likely not to be possible.  They should have no state associated with
them, however, and so parsing should be entirely reentrant.  State, for a
grammar, is in two categories:

- The current parsing state
- The actions to be performed when parsing

The actions can also be immutable (they don't change over a parse, at least),
but should not be tied to the grammar.  It should be possible to write one
singleton class encapsulating the grammar, with members for the rules, and
singleton subclasses (or, ideally, delegates) providing parser actions.  The
parsing state should be entirely contained within a context object and so the
same grammar can be used from multiple threads and can be used for compilation,
syntax highlighting, and so on.

RTTI Usage
----------

ParserLib requires RTTI for one specific purpose: down-casting from `ast_node`
to a subclass (and checking that the result really is of that class).  If you
are using RTTI in the rest of your application, then you can instruct ParserLib
to use RTTI for these casts by defining the `USE_RTTI` macro before including
the ParserLib headers and when building ParserLib.

If you do not wish to depend on RTTI, then ParserLib provides a macro that you
can use in your own AST classes that will provide the required virtual
functions to implement ad-hoc RTTI for this specific use.  You use them like
this:

{% highlight c++ %}
	class MyASTClass : parserlib::ast_node
	{
		/* Your methods go here. */
		PARSELIB_RTTI(MyASTClass, parserlib::ast_node)
	};
{% endhighlight %}

This macro will be compiled away if you do define `USE_RTTI`, so you can
provide grammars built with ParserLib that don't force consumers to use or
not-use RTTI.  It is also completely safe to build without `USE_RTTI`, but
still compile with RTTI.

What is Pegmatite
-----------------

Pegmatite is a very crystalline, intrusive igneous rock composed of
interlocking crystals usually larger than 2.5 cm in size.

It is also, in the Computer Lab's tradition of using bad puns for naming, a Parsing Expression Grammar library that rocks!

