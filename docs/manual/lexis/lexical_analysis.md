## Lexical Analysis

### Encoding

All Nim source files are in the UTF-8 encoding (or its ASCII subset).
Other encodings are not supported. Any of the standard platform line
termination sequences can be used - the Unix form using ASCII LF
(linefeed), the Windows form using the ASCII sequence CR LF (return
followed by linefeed), or the old Macintosh form using the ASCII CR
(return) character. All of these forms can be used equally, regardless
of the platform.

### Indentation

Nim\'s standard grammar describes an
`indentation sensitive` language. This
means that all the control structures are recognized by indentation.
Indentation consists only of spaces; tabulators are not allowed.

The indentation handling is implemented as follows: The lexer annotates
the following token with the preceding number of spaces; indentation is
not a separate token. This trick allows parsing of Nim with only 1 token
of lookahead.

The parser uses a stack of indentation levels: the stack consists of
integers counting the spaces. The indentation information is queried at
strategic places in the parser but ignored otherwise: The
pseudo-terminal `IND{>}` denotes an indentation that consists of more
spaces than the entry at the top of the stack; `IND{=}` an indentation
that has the same number of spaces. `DED` is another pseudo terminal
that describes the *action* of popping a value from the stack, `IND{>}`
then implies to push onto the stack.

With this notation we can now easily define the core of the grammar: A
block of statements (simplified example):

    ifStmt = 'if' expr ':' stmt
             (IND{=} 'elif' expr ':' stmt)*
             (IND{=} 'else' ':' stmt)?

    simpleStmt = ifStmt / ...

    stmt = IND{>} stmt ^+ IND{=} DED  # list of statements
         / simpleStmt                 # or a simple statement

### Comments

Comments start anywhere outside a string or character literal with the
hash character `#`. Comments consist of a concatenation of
`comment pieces`. A comment piece starts
with `#` and runs until the end of the line. The end of line characters
belong to the piece. If the next line only consists of a comment piece
with no other tokens between it and the preceding one, it does not start
a new comment:

``` nim
i = 0     # This is a single comment over multiple lines.
# The lexer merges these two pieces.
# The comment continues here.
```

`Documentation comments` are comments that
start with two `##`. Documentation comments are tokens; they are only
allowed at certain places in the input file as they belong to the syntax
tree.

### Multiline comments

Starting with version 0.13.0 of the language Nim supports multiline
comments. They look like:

``` nim
#[Comment here.
Multiple lines
are not a problem.]#
```

Multiline comments support nesting:

``` nim
#[  #[ Multiline comment in already
commented out code. ]#
proc p[T](x: T) = discard
]#
```

Multiline documentation comments also exist and support nesting too:

``` nim
proc foo =
##[Long documentation comment
here.
]##
```

### Identifiers & Keywords

Identifiers in Nim can be any string of letters, digits and underscores,
with the following restrictions:

-   begins with a letter
-   does not end with an underscore `_`
-   two immediate following underscores `__` are not allowed:

``` letter ::= 'A'..'Z' | 'a'..'z' | '\x80'..'\xff'
digit ::= '0'..'9'
IDENTIFIER ::= letter ( ['_'] (letter | digit) )*
```

Currently, any Unicode character with an ordinal value \> 127
(non-ASCII) is classified as a `letter` and may thus be part of an
identifier but later versions of the language may assign some Unicode
characters to belong to the operator characters instead.

The following keywords are reserved and cannot be used as identifiers:

``` {.nim file="keywords.txt"}
```

Some keywords are unused; they are reserved for future developments of
the language.

### Identifier equality

Two identifiers are considered equal if the following algorithm returns
true:

``` nim
proc sameIdentifier(a, b: string): bool =
a[0] == b[0] and
a.replace("_", "").toLowerAscii == b.replace("_", "").toLowerAscii
```

That means only the first letters are compared in a case-sensitive
manner. Other letters are compared case-insensitively within the ASCII
range and underscores are ignored.

This rather unorthodox way to do identifier comparisons is called
`partial case-insensitivity` and has some
advantages over the conventional case sensitivity:

It allows programmers to mostly use their own preferred spelling style,
be it humpStyle or snake_style, and libraries written by different
programmers cannot use incompatible conventions. A Nim-aware editor or
IDE can show the identifiers as preferred. Another advantage is that it
frees the programmer from remembering the exact spelling of an
identifier. The exception with respect to the first letter allows common
code like `var foo: Foo` to be parsed unambiguously.

Note that this rule also applies to keywords, meaning that `notin` is
the same as `notIn` and `not_in` (all-lowercase version (`notin`,
`isnot`) is the preferred way of writing keywords).

Historically, Nim was a fully `style-insensitive`{.interpreted-text
role="idx"} language. This meant that it was not case-sensitive and
underscores were ignored and there was not even a distinction between
`foo` and `Foo`.

### Keywords as identifiers

If a keyword is enclosed in backticks it loses its keyword property and
becomes an ordinary identifier.

Examples

``` nim
var `var` = "Hello Stropping"
```

``` nim
type Obj = object
`type`: int
let `object` = Obj(`type`: 9)
assert `object` is Obj
assert `object`.`type` == 9

var `var` = 42
let `let` = 8
assert `var` + `let` == 50

const `assert` = true
assert `assert`
```

### String literals

Terminal symbol in the grammar: `STR_LIT`.

String literals can be delimited by matching double quotes, and can
contain the following `escape sequences`:

+------------------------+--------------------------------------------+
| Escape sequence        | Meaning                                    |
+========================+============================================+
| > `\p`                 | platform specific newline: CRLF on         |
|                        | Windows, LF on Unix                        |
+------------------------+--------------------------------------------+
| > `\r`, `\c`           | `carriage return`{.interpreted-text        |
|                        | role="idx"}                                |
+------------------------+--------------------------------------------+
| > `\n`, `\l`           | `line feed`  |
|                        | (often called `newline`{.interpreted-text  |
|                        | role="idx"})                               |
+------------------------+--------------------------------------------+
| > `\f`                 | `form feed`  |
+------------------------+--------------------------------------------+
| > `\t`                 | `tabulator`  |
+------------------------+--------------------------------------------+
| > `\v`                 | `vertical tabulator`{.interpreted-text     |
|                        | role="idx"}                                |
+------------------------+--------------------------------------------+
| > `\\`                 | `backslash`  |
+------------------------+--------------------------------------------+
| > `\"`                 | `quotation mark`{.interpreted-text         |
|                        | role="idx"}                                |
+------------------------+--------------------------------------------+
| > `\'`                 | `apostrophe` |
+------------------------+--------------------------------------------+
| > `\` \'0\'..\'9\'+ \` | character with decimal value d\`:idx:; all |
|                        | decimal digits directly following are used |
|                        | for the character                          |
+------------------------+--------------------------------------------+
| > `\a`                 | `alert`      |
+------------------------+--------------------------------------------+
| > `\b`                 | `backspace`  |
+------------------------+--------------------------------------------+
| > `\e`                 | `escape`     |
|                        | `[ESC]`      |
+------------------------+--------------------------------------------+
| > `\x` HH              | `char                                      |
|                        | acter with hex value HH`{.interpreted-text |
|                        | role="idx"}; exactly two hex digits are    |
|                        | allowed                                    |
+------------------------+--------------------------------------------+
| > `\u` HHHH            | `unicode codepo                            |
|                        | int with hex value HHHH`{.interpreted-text |
|                        | role="idx"}; exactly four hex digits are   |
|                        | allowed                                    |
+------------------------+--------------------------------------------+
| > `\u` {H+}            | `unicode codepoint`{.interpreted-text      |
|                        | role="idx"}; all hex digits enclosed in    |
|                        | `{}` are used for the codepoint            |
+------------------------+--------------------------------------------+

Strings in Nim may contain any 8-bit value, even embedded zeros. However
some operations may interpret the first binary zero as a terminator.

### Triple quoted string literals

Terminal symbol in the grammar: `TRIPLESTR_LIT`.

String literals can also be delimited by three double quotes `"""` \...
`"""`. Literals in this form may run for several lines, may contain `"`
and do not interpret any escape sequences. For convenience, when the
opening `"""` is followed by a newline (there may be whitespace between
the opening `"""` and the newline), the newline (and the preceding
whitespace) is not included in the string. The ending of the string
literal is defined by the pattern `"""[^"]`, so this:

``` nim
""""long string within quotes""""
```

Produces:

    "long string within quotes"

### Raw string literals

Terminal symbol in the grammar: `RSTR_LIT`.

There are also raw string literals that are preceded with the letter `r`
(or `R`) and are delimited by matching double quotes (just like ordinary
string literals) and do not interpret the escape sequences. This is
especially convenient for regular expressions or Windows paths:

``` nim
var f = openFile(r"C:\texts\text.txt") # a raw string, so ``\t`` is no tab
```

To produce a single `"` within a raw string literal, it has to be
doubled:

``` nim
r"a""b"
```

Produces:

    a"b

`r""""` is not possible with this notation, because the three leading
quotes introduce a triple quoted string literal. `r"""` is the same as
`"""` since triple quoted string literals do not interpret escape
sequences either.

### Generalized raw string literals

Terminal symbols in the grammar: `GENERALIZED_STR_LIT`,
`GENERALIZED_TRIPLESTR_LIT`.

The construct `identifier"string literal"` (without whitespace between
the identifier and the opening quotation mark) is a generalized raw
string literal. It is a shortcut for the construct
`identifier(r"string literal")`, so it denotes a routine call with a raw
string literal as its only argument. Generalized raw string literals are
especially convenient for embedding mini languages directly into Nim
(for example regular expressions).

The construct `identifier"""string literal"""` exists too. It is a
shortcut for `identifier("""string literal""")`.

### Character literals

Character literals are enclosed in single quotes `''` and can contain
the same escape sequences as strings - with one exception: the platform
dependent `newline` (`\p`) is not allowed
as it may be wider than one character (it can be the pair CR/LF). Here
are the valid `escape sequences` for
character literals:

+---------------------+-----------------------------------------------+
| Escape sequence     | Meaning                                       |
+=====================+===============================================+
| > `\r`, `\c`        | `carriage return`{.interpreted-text           |
|                     | role="idx"}                                   |
+---------------------+-----------------------------------------------+
| > `\n`, `\l`        | `line feed`     |
+---------------------+-----------------------------------------------+
| > `\f`              | `form feed`     |
+---------------------+-----------------------------------------------+
| > `\t`              | `tabulator`     |
+---------------------+-----------------------------------------------+
| > `\v`              | `vertical tabulator`{.interpreted-text        |
|                     | role="idx"}                                   |
+---------------------+-----------------------------------------------+
| > `\\`              | `backslash`     |
+---------------------+-----------------------------------------------+
| > `\"`              | `quotation mark`{.interpreted-text            |
|                     | role="idx"}                                   |
+---------------------+-----------------------------------------------+
| > `\'`              | `apostrophe`    |
+---------------------+-----------------------------------------------+
| > `\` \'0\'..\'9\'+ | `char                                         |
|                     | acter with decimal value d`{.interpreted-text |
|                     | role="idx"}; all decimal digits directly      |
|                     | following are used for the character          |
+---------------------+-----------------------------------------------+
| > `\a`              | `alert`         |
+---------------------+-----------------------------------------------+
| > `\b`              | `backspace`     |
+---------------------+-----------------------------------------------+
| > `\e`              | `escape`        |
|                     | `[ESC]`         |
+---------------------+-----------------------------------------------+
| > `\x` HH           | `c                                            |
|                     | haracter with hex value HH`{.interpreted-text |
|                     | role="idx"}; exactly two hex digits are       |
|                     | allowed                                       |
+---------------------+-----------------------------------------------+

A character is not a Unicode character but a single byte.

Rationale: It enables the efficient support of `array[char, int]` or
`set[char]`.

The `Rune` type can represent any Unicode character. `Rune` is declared
in the [unicode module](unicode.html).

A character literal that does not end in `'` is interpreted as `'` if
there is a preceeding backtick token. There must be no whitespace
between the preceeding backtick token and the character literal. This
special case ensures that a declaration like
`` proc `'customLiteral`(s: string) `` is valid.
`` proc `'customLiteral`(s: string) `` is the same as
`` proc `'\''customLiteral`(s: string) ``.

See also [Custom Numeric Literals](#custom-numeric-literals).

### Numeric Literals

Numeric literals have the form:

    hexdigit = digit | 'A'..'F' | 'a'..'f'
    octdigit = '0'..'7'
    bindigit = '0'..'1'
    unary_minus = '-' # See the section about unary minus
    HEX_LIT = unary_minus? '0' ('x' | 'X' ) hexdigit ( ['_'] hexdigit )*
    DEC_LIT = unary_minus? digit ( ['_'] digit )*
    OCT_LIT = unary_minus? '0' 'o' octdigit ( ['_'] octdigit )*
    BIN_LIT = unary_minus? '0' ('b' | 'B' ) bindigit ( ['_'] bindigit )*

    INT_LIT = HEX_LIT
            | DEC_LIT
            | OCT_LIT
            | BIN_LIT

    INT8_LIT = INT_LIT ['\''] ('i' | 'I') '8'
    INT16_LIT = INT_LIT ['\''] ('i' | 'I') '16'
    INT32_LIT = INT_LIT ['\''] ('i' | 'I') '32'
    INT64_LIT = INT_LIT ['\''] ('i' | 'I') '64'

    UINT_LIT = INT_LIT ['\''] ('u' | 'U')
    UINT8_LIT = INT_LIT ['\''] ('u' | 'U') '8'
    UINT16_LIT = INT_LIT ['\''] ('u' | 'U') '16'
    UINT32_LIT = INT_LIT ['\''] ('u' | 'U') '32'
    UINT64_LIT = INT_LIT ['\''] ('u' | 'U') '64'

    exponent = ('e' | 'E' ) ['+' | '-'] digit ( ['_'] digit )*
    FLOAT_LIT = unary_minus? digit (['_'] digit)* (('.' digit (['_'] digit)* [exponent]) |exponent)
    FLOAT32_SUFFIX = ('f' | 'F') ['32']
    FLOAT32_LIT = HEX_LIT '\'' FLOAT32_SUFFIX
                | (FLOAT_LIT | DEC_LIT | OCT_LIT | BIN_LIT) ['\''] FLOAT32_SUFFIX
    FLOAT64_SUFFIX = ( ('f' | 'F') '64' ) | 'd' | 'D'
    FLOAT64_LIT = HEX_LIT '\'' FLOAT64_SUFFIX
                | (FLOAT_LIT | DEC_LIT | OCT_LIT | BIN_LIT) ['\''] FLOAT64_SUFFIX

    CUSTOM_NUMERIC_LIT = (FLOAT_LIT | INT_LIT) '\'' CUSTOM_NUMERIC_SUFFIX

    # CUSTOM_NUMERIC_SUFFIX is any Nim identifier that is not
    # a pre-defined type suffix.

As can be seen in the productions, numeric literals can contain
underscores for readability. Integer and floating-point literals may be
given in decimal (no prefix), binary (prefix `0b`), octal (prefix `0o`),
and hexadecimal (prefix `0x`) notation.

The fact that the unary minus `-` in a number literal like `-1` is
considered to be part of the literal is a late addition to the language.
The rationale is that an expression `-128'i8` should be valid and
without this special case, this would be impossible \-- `128` is not a
valid `int8` value, only `-128` is.

For the `unary_minus` rule there are further restrictions that are not
covered in the formal grammar. For `-` to be part of the number literal
its immediately preceeding character has to be in the set
`{' ', '\t', '\n', '\r', ',', ';', '(', '[', '{'}`. This set was
designed to cover most cases in a natural manner.

In the following examples, `-1` is a single token:

``` nim
echo -1
echo(-1)
echo [-1]
echo 3,-1

"abc";-1
```

In the following examples, `-1` is parsed as two separate tokens (as
`-`{.interpreted-text role="tok"} `1`{.interpreted-text role="tok"}):

``` nim
echo x-1
echo (int)-1
echo [a]-1
"abc"-1
```

The suffix starting with an apostrophe (\'\'\') is called a
`type suffix`. Literals without a type
suffix are of an integer type unless the literal contains a dot or `E|e`
in which case it is of type `float`. This integer type is `int` if the
literal is in the range `low(int32)..high(int32)`, otherwise it is
`int64`. For notational convenience, the apostrophe of a type suffix is
optional if it is not ambiguous (only hexadecimal floating-point
literals with a type suffix can be ambiguous).

The pre-defined type suffixes are:

+-------------+---------------------------+
| Type Suffix | Resulting type of literal |
+=============+===========================+
| > `'i8`     | int8                      |
+-------------+---------------------------+
| > `'i16`    | int16                     |
+-------------+---------------------------+
| > `'i32`    | int32                     |
+-------------+---------------------------+
| > `'i64`    | int64                     |
+-------------+---------------------------+
| > `'u`      | uint                      |
+-------------+---------------------------+
| > `'u8`     | uint8                     |
+-------------+---------------------------+
| > `'u16`    | uint16                    |
+-------------+---------------------------+
| > `'u32`    | uint32                    |
+-------------+---------------------------+
| > `'u64`    | uint64                    |
+-------------+---------------------------+
| > `'f`      | float32                   |
+-------------+---------------------------+
| > `'d`      | float64                   |
+-------------+---------------------------+
| > `'f32`    | float32                   |
+-------------+---------------------------+
| > `'f64`    | float64                   |
+-------------+---------------------------+

Floating-point literals may also be in binary, octal or hexadecimal
notation:
`0B0_10001110100_0000101001000111101011101111111011000101001101001001'f64`
is approximately 1.72826e35 according to the IEEE floating-point
standard.

Literals must match the datatype, for example, `333'i8` is an invalid
literal. Non-base-10 literals are used mainly for flags and bit pattern
representations, therefore the checking is done on bit width and not on
value range. Hence: 0b10000000\'u8 == 0x80\'u8 == 128, but,
0b10000000\'i8 == 0x80\'i8 == -1 instead of causing an overflow error.

#### Custom Numeric Literals

If the suffix is not predefined, then the suffix is assumed to be a call
to a proc, template, macro or other callable identifier that is passed
the string containing the literal. The callable identifier needs to be
declared with a special `'` prefix:

``` nim
import strutils
type u4 = distinct uint8 # a 4-bit unsigned integer aka "nibble"
proc `'u4`(n: string): u4 =
  # The leading ' is required.
  result = (parseInt(n) and 0x0F).u4

var x = 5'u4
```

More formally, a custom numeric literal `123'custom` is transformed to
r\"123\".`'custom` in the parsing step. There is no AST node kind that
corresponds to this transformation. The transformation naturally handles
the case that additional parameters are passed to the callee:

``` nim
import strutils
type u4 = distinct uint8 # a 4-bit unsigned integer aka "nibble"
proc `'u4`(n: string; moreData: int): u4 =
  result = (parseInt(n) and 0x0F).u4

var x = 5'u4(123)
```

Custom numeric literals are covered by the grammar rule named
`CUSTOM_NUMERIC_LIT`. A custom numeric literal is a single token.

### Operators

Nim allows user defined operators. An operator is any combination of the
following characters:

    =     +     -     *     /     <     >
    @     $     ~     &     %     |
    !     ?     ^     .     :     \

(The grammar uses the terminal OPR to refer to operator symbols as
defined here.)

These keywords are also operators:
`and or not xor shl shr div mod in notin is isnot of as from`.

`.`{.interpreted-text role="tok"}, `=`{.interpreted-text role="tok"},
`:`{.interpreted-text role="tok"}, `::`{.interpreted-text role="tok"}
are not available as general operators; they are used for other
notational purposes.

`*:` is as a special case treated as the two tokens
`*`{.interpreted-text role="tok"} and `:`{.interpreted-text role="tok"}
(to support `var v*: T`).

The `not` keyword is always a unary operator, `a not b` is parsed as
`a(not b)`, not as `(a) not (b)`.

### Other tokens

The following strings denote other tokens:

    `   (    )     {    }     [    ]    ,  ;   [.    .]  {.   .}  (.  .)  [:

The `slice` operator
`..`{.interpreted-text role="tok"} takes precedence over other tokens
that contain a dot: `{..}` are the three tokens `{`{.interpreted-text
role="tok"}, `..`{.interpreted-text role="tok"}, `}`{.interpreted-text
role="tok"} and not the two tokens `{.`{.interpreted-text role="tok"},
`.}`{.interpreted-text role="tok"}.

### Unicode Operators

Under the `--experimental:unicodeOperators` switch these Unicode
operators are also parsed as operators:

    ∙ ∘ × ★ ⊗ ⊘ ⊙ ⊛ ⊠ ⊡ ∩ ∧ ⊓   # same priority as * (multiplication)
    ± ⊕ ⊖ ⊞ ⊟ ∪ ∨ ⊔             # same priority as + (addition)

If enabled, Unicode operators can be combined with non-Unicode operator
symbols. The usual precedence extensions then apply, for example, `⊠=`
is an assignment like operator just like `*=` is.

No Unicode normalization step is performed.

**Note**: Due to parser limitations one **cannot** enable this feature
via a pragma `{.experimental: "unicodeOperators".}` reliably.


