Many ECMAScript features, both adopted and proposed, make use of contextual keywords.  These produce interactions, often unintended, with other proposals.
Here we look at cross-cutting steps we can make to ameliorate those and reduce the language cognitive burden.
This section is inspired by some of the discussion in [proposal-pattern-matching#322](https://github.com/tc39/proposal-pattern-matching/issues/322).

### Terminology

We'll use the following terminology here, which differs slightly from [ECMAScript's definitions](https://tc39.es/ecma262/#sec-keywords-and-reserved-words).
* A ***word*** (called [*IdentifierName*](https://tc39.es/ecma262/#prod-IdentifierName) in ECMAScript) is a lexical token that starts with a letter or underscore and continues with a sequence of letters, underscores, or digits.
* An ***identifier*** is a *word* used to name variables, functions, classes, property names, labels, and such, which we'll collectively refer to as ***variables***.
* A ***keyword*** is a *word* used as a piece of syntax instead of naming a variable: `if`, `async`, `yield`. *Keywords* can be either *reserved keywords* or *contextual keywords*.
  * A ***reserved keyword*** is a *keyword* that cannot be used as an *identifier* in its *neighborhood*.
  * A ***contextual keyword*** is *keyword* that can be used as an *identifier* in its *neighborhood*.
* A ***non-reserved word*** is either a *contextual keyword* or an *identifier*.
* A ***neighborhood*** is either the whole script or a well-defined scope that's readily apparent to users.

Whether a keyword is reserved or contextual depends on its neighborhood. Let's give some examples:
* In a few neighborhoods, such as after a `.` or in some parts of object literals, no keywords at all are present.  There are no reserved keywords and no contextual keywords.  All words are identifiers — `if` can be used as an identifier in those neighborhoods. 
* Keywords such as `if` and `class` are reserved keywords in all neighborhoods other than the above. These are never used as contextual keywords.
* Keywords such as `yield` and `await` are reserved keywords inside generators and async functions respectively.  They are never used as contextual keywords.  Outside those neighborhoods they are identifiers.
* Keywords such as `let` are reserved keywords in strict-mode neighborhoods but contextual keywords in non-strict-mode neighborhoods.
* Keywords such as `async` are contextual keywords.

### Prefix vs. Infix

Many features, both existing and proposed, assign various meanings to juxtapositions of non-reserved words.  A common pattern is:

```
nonReservedWord1 nonReservedWord2 somethingElse
```
