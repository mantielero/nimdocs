## Syntax

This section lists Nim\'s standard syntax. How the parser handles the
indentation is already described in the [Lexical
Analysis](#lexical-analysis) section.

Nim allows user-definable operators. Binary operators have 11 different
levels of precedence.

### Associativity

Binary operators whose first character is `^` are right-associative, all
other binary operators are left-associative.

``` nim
proc `^/`(x, y: float): float =
# a right-associative division operator
result = x / y
echo 12 ^/ 4 ^/ 8 # 24.0 (4 / 8 = 0.5, then 12 / 0.5 = 24.0)
echo 12  / 4  / 8 # 0.375 (12 / 4 = 3.0, then 3 / 8 = 0.375)
```

### Precedence

Unary operators always bind stronger than any binary operator: `$a + b`
is `($a) + b` and not `$(a + b)`.

If a unary operator\'s first character is `@` it is a
`sigil-like` operator which binds stronger
than a \`primarySuffix\`: `@x.abc` is parsed as `(@x).abc` whereas
`$x.abc` is parsed as `$(x.abc)`.

For binary operators that are not keywords, the precedence is determined
by the following rules:

Operators ending in either `->`, `~>` or `=>` are called
`arrow like`, and have the lowest
precedence of all operators.

If the operator ends with `=` and its first character is none of `<`,
`>`, `!`, `=`, `~`, `?`, it is an *assignment operator* which has the
second-lowest precedence.

Otherwise, precedence is determined by the first character.

+----------------+----------------+----------------+----------------+
| Precedence     | Operators      | First          | Terminal       |
| level          |                | character      | symbol         |
+================+================+================+================+
| > 10 (highest) |                | `$  ^`         | OP10           |
+----------------+----------------+----------------+----------------+
| > 9            | `*    /        | `*  %  \  /`   | OP9            |
|                |     div   mod  |                |                |
|                |   shl  shr  %` |                |                |
+----------------+----------------+----------------+----------------+
| > 8            | `+    -`       | `+  -  ~  |`   | OP8            |
+----------------+----------------+----------------+----------------+
| > 7            | `&`            | `&`            | OP7            |
+----------------+----------------+----------------+----------------+
| > 6            | `..`           | `.`            | OP6            |
+----------------+----------------+----------------+----------------+
| > 5 4 3 2 1 0  | `==  <= <      | `=  <  >  !`   | OP5 OP4 OP3    |
| > (lowest)     | >= > !=  in no |                | OP2 OP1 OP0    |
|                | tin is isnot n | `@  :  ?`      |                |
|                | ot of as from` |                |                |
|                | `and` `or xor` |                |                |
|                |                |                |                |
|                | *assignment    |                |                |
|                | operator*      |                |                |
|                | (like `+=`,    |                |                |
|                | `*=`) *arrow   |                |                |
|                | like operator* |                |                |
|                | (like `->`,    |                |                |
|                | `=>`)          |                |                |
+----------------+----------------+----------------+----------------+

Whether an operator is used as a prefix operator is also affected by
preceding whitespace (this parsing change was introduced with version
0.13.0):

``` nim
echo $foo
# is parsed as
echo($foo)
```

Spacing also determines whether `(a, b)` is parsed as an argument list
of a call or whether it is parsed as a tuple constructor:

``` nim
echo(1, 2) # pass 1 and 2 to echo
```

``` nim
echo (1, 2) # pass the tuple (1, 2) to echo
```

### Dot-like operators

Terminal symbol in the grammar: `DOTLIKEOP`.

Dot-like operators are operators starting with `.`, but not with `..`,
for e.g. `.?`; they have the same precedence as `.`, so that `a.?b.c` is
parsed as `(a.?b).c` instead of `a.?(b.c)`.

### Grammar

The grammar\'s start symbol is `module`.

``` {. literal=""}
```
