---
layout: default
title: Defining a grammar
next_section: astbuild
prev_section: index
---

Pegmatite grammars are [Parsing Expression
Grammars](https://en.wikipedia.org/wiki/Parsing_expression_grammar) (PEGs).  As
such, they are always matched greedily and there is no ambiguity in the
grammar: the first rule to match is always the one taken.

Traditional PEGs do not support left recursion, because greedy matching of a
left-recursive rule would simply involve it repeatedly trying to match the left
recursive part until it ran out of stack space.
[OMeta](https://en.wikipedia.org/wiki/OMeta) added support for left recursion,
as part of its parsing algorithm.  Pegmatite uses a simplified form of the
OMeta algorithm, which has worse worst-case running time, but better
worst-case space usage.

Once left recursion is encountered, including indirect left recursion, the
parser will backtrack until it finds a conditional that has a branch that is
not left recursive.  

You can create freestanding `Rule` instances, however the recommended way of
creating a grammar is to make each rule a field in a class.  The Calculator
example follows this pattern, with a `CalculatorGrammar` class containing one
field for each rule.  This is recommended for two reasons:

 1. It allows lazy creation, rather than requiring the rules to be instantiated
    on program creation.  This gives faster start-up times and means that
    memory is only used when the grammar is actually used.
 2. It improves encapsulation.  Although it is completely safe to refer to
    rules in other grammars, placing them in a class makes it easy for people
    binding actions to the grammar to identify all of the rules that they must
    (or, might want to) provide actions for.

Simple character or string recognising expressions can be created using the
`_E` custom literal suffix.  For example `"int"_E` creates an expression that
will match the literal string "int".  This is useful for terminals.  The `_S`
suffix defines a set, for example `"eE"_S` matches either the character 'e' or
'E'.

You can define more complex operations from these by using the following
operators (where `a` and `b` are expressions):

 - `*a` matches zero or more instances of `a`
 - `+a` matches one or more instances of `a`
 - `-a` matches zero or one instance of `a`
 - `a >> b` matches `a` and then `b`
 - `a | b` matches either `a` or `b`

Newline rules have no special meaning in parsing, but are used to increment the
line counter for input ranges.  If you declare a rule as matching a newline, it
will increment the line counter every time it is successfully matched.

Whitespace rules allow implicit whitespace in between all non-terminal
expressions (sequences).

In Pegmatite, the grammar is kept separate from the set of actions that are
invoked when matching it.  The grammar from the calculator example is shown below:

{% highlight c++ %}
struct CalculatorGrammar
{
	/**
	 * Only spaces are recognised as whitespace in this toy example.
	 */
	Rule ws     = " \t\n"_E;
	/**
	 * Digits are things in the range 0-9.
	 */
	Rule digit  = '0'_E - '9';
	/**
	 * Numbers are one or more digits, optionally followed by a decimal point,
	 * and one or more digits, optionally followed by an exponent (which may
	 * also be negative.
	 */
	Rule num    = +digit >> -('.'_E >> +digit >> -("eE"_S >> -("+-"_S) >> +digit));
	/**
	 * Values are either numbers or expressions in brackets (highest precedence).
	 */
	Rule val    = num |  '(' >> expr >> ')';
	/**
	 * Multiply operations are values or multiply, or divide operations,
	 * followed by a multiply symbol, followed by a value.  The sides can never
	 * be add or subtract operations, because they have lower precedence and so
	 * can only be parents of multiply or divide operations (or children via
	 * parenthetical expressions), not direct children.
	 */
	Rule mul_op = mul >> '*' >> val;
	/**
	 * Divide operations follow the same syntax as multiply.
	 */
	Rule div_op = mul >> '/' >> val;
	/**
	 * Multiply-precedence operations are either multiply or divide operations,
	 * or simple values (numbers of parenthetical expressions).
	 */
	Rule mul    = mul_op | div_op | val;
	/**
	 * Add operations can have any expression on the left (including other add
	 * expressions), but only higher-precedence operations on the right.
	 */
	Rule add_op = expr >> '+' >> expr;
	/**
	 * Subtract operations follow the same structure as add.
	 */
	Rule sub_op = expr >> '-' >> expr;
	/**
	 * Expressions can be any of the other types.
	 */
	Rule expr   = add_op | sub_op | mul;

	/**
	 * Returns a singleton instance of this grammar.
	 */
	static const CalculatorGrammar& get()
	{
		static CalculatorGrammar g;
		return g;
	}
	private:
	/**
	 * Private constructor.  This class is immutable, and so only the `get()`
	 * method should be used to return the singleton instance.
	 */
	CalculatorGrammar() {};
};
{% endhighlight %}

Note that the `_E` suffix is only required for the first element in an expression.  For example, `'0'_E - '9'` defines a range expression that will match any character in the range 0--9.  The suffix is required on the first literal, but the argument to the `-` operator (which defines a range) will accept either another expression or a literal character.

This can be seen in other sequences, for example this rule defines subtract operations:

{% highlight c++ %}
	Rule sub_op = expr >> '-' >> expr;
{% endhighlight %}

The literals here do not need the suffix, because they are all on the right-hand side of an operator that defines a compound expression.

This grammar is a singleton and can be reused by multiple consumers.
