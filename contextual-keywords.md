Many ECMAScript features, both adopted and proposed, make use of contextual keywords.  These produce interactions, often unintended, with other proposals.
Here we look at cross-cutting steps we can make to ameliorate those and reduce the language cognitive burden.
This section is inspired by some of the discussion in [proposal-pattern-matching#322](https://github.com/tc39/proposal-pattern-matching/issues/322).

## Terminology

We'll use the following terminology here, which differs slightly from [ECMAScript's definitions](https://tc39.es/ecma262/#sec-keywords-and-reserved-words).
* A ***word*** (called [*IdentifierName*](https://tc39.es/ecma262/#prod-IdentifierName) in ECMAScript) is a lexical token that starts with a letter or underscore and continues with a sequence of letters, underscores, or digits.
* An ***identifier*** is a *word* used to name variables, functions, classes, property names, labels, and such, which we'll collectively refer to as ***variables***.
* A ***keyword*** is a *word* used as a piece of syntax instead of naming a variable: `if`, `async`, `yield`. *Keywords* can be either *reserved keywords* or *contextual keywords*.
  * A ***reserved keyword*** is a *keyword* that cannot be used as an *identifier* in its *neighborhood*.
  * A ***contextual keyword*** is *keyword* that can be used as an *identifier* in its *neighborhood*.
* A ***non-reserved word*** is either a *contextual keyword* or an *identifier*.
* A ***neighborhood*** is either the whole script or a well-defined scope that's readily apparent to and must be understood by users to make effective use of the language.

Whether a keyword is reserved or contextual depends on its neighborhood. Let's give some examples:
* In a few neighborhoods, such as after a `.` or in some parts of object literals, no keywords at all are present.  There are no reserved keywords and no contextual keywords.  All words are identifiers — `if` can be used as an identifier in those neighborhoods. 
* Keywords such as `if` and `class` are reserved keywords in all neighborhoods other than the above. These are never used as contextual keywords.
* Keywords such as `yield` and `await` are reserved keywords inside generators and async functions respectively.  They are never used as contextual keywords.  Outside those neighborhoods they are identifiers.
* Keywords such as `let` are reserved keywords in strict-mode neighborhoods but contextual keywords in non-strict-mode neighborhoods.
* Keywords such as `async` are contextual keywords.

## Safe Contextual Keywords

Some contextual keywords appear in grammatical contexts where nothing else can appear; examples are `target` and `meta` in `new . target` and `import . meta` respectively. There's nothing other than a contextual keyword that can follow `new .` or `import .`.  These are safe and future-proof and do not induce conflicts except in the unlikely case were we to introduce a meaning to <code>new . *identifier*</code> or <code>import . *identifier*</code> in the future.

Contextual keywords such as `from` are somewhere in between safe contextual keywords and the ones below, depending on the specifics of how `import` syntax evolves.

## Prefix vs. Infix Contextual Keywords

It's highly desirable to introduce new language syntax features without breaking backward compatibility by reserving new keywords.  Many such features, both existing and proposed, assign various meanings to juxtapositions of non-reserved words in places where expressions or statements can also occur.  A common pattern is:

<pre>
<i>nonReservedWord1</i> <i>nonReservedWord2</i> <i>somethingElse</i>
</pre>

In earlier days of ECMAScript before contextual keywords came into use, this was a syntax error as long as there was no line break between *nonReservedWord1* and *nonReservedWord2*. If there was a line break there, this turned into a one-word expression statement *nonReservedWord1* followed by another expression statement or statements starting with *nonReservedWord2*.

With such a blank slate one could take one of two directions:

1. Gradually add new features using a prefix contextual keyword: <code><i>contextualKeyword</i> <i>identifier</i> <i>somethingElse</i></code>
2. Gradually add new features using an infix or suffix contextual keyword: <code><i>identifier</i> <i>contextualKeyword</i> <i>somethingElse</i></code>

We could add arbitrarily many new features using approach 1 without conflicts.  Alternatively, we could add arbitrarily many new features using approach 2 without conflicts.  But because a contextual keyword can also be used as an identifier, we can't do both without them colliding in user-visible ways, with the number of collisions and special cases growing quadratically.  If proposals appear gradually, when parsing <code><i>contextualKeywordA</i> <i>contextualKeywordB</i> <i>somethingElse</i></code>, we'd have to let whichever of *contextualKeywordA* or *contextualKeywordB* happened to be standardized first win and treat the other as an identifier, which is a form of shipping our org chart over time.

## Examples

### Prefix

The bulk of the relevant contextual keywords proposed or added to ECMAScript so far are of the prefix variety.  Some examples:

<pre>
let <i>identifier</i> …
async <i>identifier</i> …
using <i>identifier</i> …
</pre>

Interestingly, we allowed line breaks after `let` but not after the other two.  This means that one can write a let-statement as:

```
let
  identifier1 = expr1,
  identifier2 = expr2,
  identifier3 = expr3;
```

but 

```
using
  identifier1 = expr1,
  identifier2 = expr2,
  identifier3 = expr3;
```

is just an unused expression-statement containing an identifier named `using` followed by a comma-expression containing three regular assignments.

If that were the entirety of the situation, we could add arbitrarily more prefix contextual keywords without too much trouble.

### Infix

We also added one infix contextual keyword:

<pre>
<i>identifier</i> of …
</pre>

Even though it's limited to the inside of some for-loops, this has caused problems.  We had to fix bugs caused by

```
  for (async of (x)…
```

which could have meant either a for-of iteration with `async` as a variable name or a regular for-loop starting with an async function named `of`.

`of` was added before `using`, which necessitated workarounds for a growing number of cases:

```
  for (using of …)
```

and the [fascinating](https://github.com/tc39/proposal-explicit-resource-management/issues/107)

```
  for (using of of [x])
```

### A Choice

There's a [current proposal](https://github.com/tc39/proposal-pattern-matching) to add a new kind of infix expression operator and a debate about whether to name it `~=` or `is`.
The `~=` would be a reserved token and cause no significant conflicts.  On the other hand, using a contextual keyword would have some rather far-reaching consequences:

```
let is
expr ?
foo : bar;
```

is existing syntax.  This could be fixed by prohibiting line breaks on *both* sides of `is`, but restricting line breaks is not free.  The right operand of `is` can often be long, and prohibiting line breaks on both sides of
a general infix operator should be avoided if at all possible.

The grammar on the right side of `is` tokenizes and parses the same text as expressions in very different ways.  This creates plenty of opportunity for the lexer's interpretation of `/` vs. `/regexp/` to diverge.
Here is a neat example using the current pattern matching grammar:

```
for (using is of and [not/a+"/g]; b++; [/"/g, 5])
```

Here `using`, `is`, `of`, `and`, and `not` are all non-reserved words.

* For each of those non-reserved words, there is a parse of the above statement that treats it as a contextual keyword.  There is another parse that treats it as an identifier.
* For each `"` in the text, there is a parse that treats it as delimiting a string.  There is another parse that treats it as a quote character.
* For each `/` in the text, there is a parse that treats it as a division.  There is another parse that treats it as the start or end of a regular expression literal.
* For each `;` in the text, there is a parse that treats it as a for-loop header separator.  There is another parse that treats it as a string character.

There are also other cases.  It's *very* difficult to find them all without automated validators that understand multi-lookahead cover grammars.

This is not a problem that can be solved once.  If `is` is adopted as an infix contextual keyword, every future proposal that tries to add a new prefix contextual keyword will have to deal with new such cases.

The `~=` alternative syntax has none of these problems:

```
for (using ~= of and [not/a+"/g]; b++; [/"/g, 5])
```

is unambiguous.  And, unlike `is`, we can add line breaks before or after `~=` without any difficulties.
