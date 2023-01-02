## About this document
[original](https://nim-lang.org/docs/manual.html#about-this-document)

**Note**: This document is a draft! Several of Nim's features may need
more precise wording. This manual is constantly evolving into a proper
specification.

**Note**: The experimental features of Nim are covered
[here](https://nim-lang.org/docs/manual_experimental.html).

**Note**: Assignments, moves, and destruction are specified in the
[destructors](https://nim-lang.org/docs/destructors.html) document.

This document describes the lexis, the syntax, and the semantics of the
Nim language.

To learn how to compile Nim programs and generate documentation see the
[Compiler User Guide](https://nim-lang.org/docs/nimc.html) and the [DocGen Tools
Guide](https://nim-lang.org/docs/docgen.html).

The language constructs are explained using an extended BNF, in which
`(a)*` means 0 or more `a`'s, `a+` means 1 or more `a`'s, and `(a)?`
means an optional *a*. Parentheses may be used to group elements.

`&` is the lookahead operator; `&a` means that an `a` is expected but
not consumed. It will be consumed in the following rule.

The `|`, `/` symbols are used to mark alternatives and have the lowest
precedence. `/` is the ordered choice that requires the parser to try
the alternatives in the given order. `/` is often used to ensure the
grammar is not ambiguous.

Non-terminals start with a lowercase letter, abstract terminal symbols
are in UPPERCASE. Verbatim terminal symbols (including keywords) are
quoted with `'`. An example:
```
ifStmt = 'if' expr ':' stmts ('elif' expr ':' stmts)* ('else' stmts)?
```

The binary `^*` operator is used as a shorthand for 0 or more
occurrences separated by its second argument; likewise `^+` means 1 or
more occurrences: `a ^+ b` is short for `a (b a)*` and `a ^* b` is short
for `(a (b a)*)?`. Example:
```
arrayConstructor = '[' expr ^* ',' ']'
```
Other parts of Nim, like scoping rules or runtime semantics, are
described informally.
