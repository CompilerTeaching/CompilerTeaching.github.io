---
layout: default
title: Building an AST
prev_section: grammar
---

The `ASTContainer` class is intended to be used as the superclass for most AST
nodes.  Any `ASTPtr` and `ASTList` fields of subclasses of this class will
automatically be created from the AST stack.  Each AST class that is
constructed is pushed onto the stack in the order that it is constructed and
then popped off by its parents.

AST nodes do not, by default, keep around the `InputRange` of the text that
they matched.  This is to save space for cases where it is not required.  If
you intend to do helpful error reporting after semantic analysis, then it is
strongly recommended that you do keep such a reference.

After you have defined your grammar and AST, all that remains is to bind the
two together.  To do this, create a subclass of `ASTParserDelegate`, with one
`BindAST` field for each AST node, initialised with the corresponding grammar
rule.  The following is the parser for the Calculator example:

{% highlight c++ %}
	class CalculatorParser : public ASTParserDelegate
	{
		BindAST<AST::Number> num = CalculatorGrammar::get().num;
		BindAST<AST::AddExpression> add = CalculatorGrammar::get().add_op;
		BindAST<AST::SubtractExpression> sub = CalculatorGrammar::get().sub_op;
		BindAST<AST::MultiplyExpression> mul = CalculatorGrammar::get().mul_op;
		BindAST<AST::DivideExpression> div = CalculatorGrammar::get().div_op;
	public:
		const CalculatorGrammar &g = CalculatorGrammar::get();
	};
{% endhighlight %}

Invoking the `parse()` method on this class will cause an `AST::Number` class
to be created for every terminal matching the `num` rule in the grammar, and so
on.  Note that this parser is reentrant.  It is safe to use it from multiple
threads to parse different strings.  It is therefore safe to also make the
parser a singleton.
