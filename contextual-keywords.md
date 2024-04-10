Many ECMAScript features, both adopted and proposed, make use of contextual keywords.  These produce interactions, often unintended, with other proposals.
Here we look at cross-cutting steps we can make to ameliorate those and reduce the language cognitive burden.
This section is inspired by some of the discussion in by some of the discussion in [proposal-pattern-matching#322](https://github.com/tc39/proposal-pattern-matching/issues/322).

### Terminology

We'll use the following terminology here, which differs slightly from [ECMAScript's definitions](https://tc39.es/ecma262/#sec-keywords-and-reserved-words).
* An *identifier* has its common meaning in the language and is used to name variables, functions, classes, property names, labels, and such, which we'll collectively refer to as *variables*.
* A *keyword* looks like an identifier but is used as a piece of syntax instead of naming a variable: `if`, `async`, `yield`. Keywords can be either *reserved keywords* or *contextual keywords*.
  * A *reserved keyword* is a *keyword* that cannot be used as an *identifier* in its neighborhood.
  * A *contextual keyword* is *keyword* that can be used as an *identifier* in its neighborhood.
* A *neighborhood* is either the whole script or a well-defined scope that's readily apparent to users.

Whether a keyword is reserved or contextual depends on its neighborhood. Let's give some examples:
* In a few neighborhoods, such as after a `.` or in some parts of object literals, no keywords at all are present.  There are no reserved keywords and no contextual keywords.  `if` can be used as an identifier in those neighborhoods. 
* Keywords such as `if` and `class` are reserved keywords in all neighborhoods other than the above. These are never used as contextual keywords.
* Keywords such as `yield` and `await` are reserved keywords inside generators and async functions respectively.  They are never used as contextual keywords.  Outside those neighborhoods they are identifiers.
* Keywords such as `let` are reserved keywords in strict-mode neighborhoods but contextual keywords in non-strict-mode neighborhoods.
* Keywords such as `async` are contextual keywords.

### Prefix vs. Infix

Many features, both existing and proposed, assign various meanings to juxtapositions of identifiers.
