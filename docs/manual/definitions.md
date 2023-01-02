## Definitions

Nim code specifies a computation that acts on a memory consisting of
components called `locations`{.interpreted-text role="idx"}. A variable
is basically a name for a location. Each variable and location is of a
certain `type`{.interpreted-text role="idx"}. The variable\'s type is
called `static type`{.interpreted-text role="idx"}, the location\'s type
is called `dynamic type`{.interpreted-text role="idx"}. If the static
type is not the same as the dynamic type, it is a super-type or subtype
of the dynamic type.

An `identifier`{.interpreted-text role="idx"} is a symbol declared as a
name for a variable, type, procedure, etc. The region of the program
over which a declaration applies is called the `scope`{.interpreted-text
role="idx"} of the declaration. Scopes can be nested. The meaning of an
identifier is determined by the smallest enclosing scope in which the
identifier is declared unless overloading resolution rules suggest
otherwise.

An expression specifies a computation that produces a value or location.
Expressions that produce locations are called
`l-values`{.interpreted-text role="idx"}. An l-value can denote either a
location or the value the location contains, depending on the context.

A Nim `program`{.interpreted-text role="idx"} consists of one or more
text `source files`{.interpreted-text role="idx"} containing Nim code.
It is processed by a Nim `compiler`{.interpreted-text role="idx"} into
an `executable`{.interpreted-text role="idx"}. The nature of this
executable depends on the compiler implementation; it may, for example,
be a native binary or JavaScript source code.

In a typical Nim program, most of the code is compiled into the
executable. However, some of the code may be executed at
`compile-time`{.interpreted-text role="idx"}. This can include constant
expressions, macro definitions, and Nim procedures used by macro
definitions. Most of the Nim language is supported at compile-time, but
there are some restrictions \-- see [Restrictions on Compile-Time
Execution](#restrictions-on-compileminustime-execution) for details. We
use the term `runtime`{.interpreted-text role="idx"} to cover both
compile-time execution and code execution in the executable.

The compiler parses Nim source code into an internal data structure
called the `abstract syntax tree`{.interpreted-text role="idx"}
(`AST`{.interpreted-text role="idx"}). Then, before executing the code
or compiling it into the executable, it transforms the AST through
`semantic analysis`{.interpreted-text role="idx"}. This adds semantic
information such as expression types, identifier meanings, and in some
cases expression values. An error detected during semantic analysis is
called a `static error`{.interpreted-text role="idx"}. Errors described
in this manual are static errors when not otherwise specified.

A `panic`{.interpreted-text role="idx"} is an error that the
implementation detects and reports at runtime. The method for reporting
such errors is via *raising exceptions* or *dying with a fatal error*.
However, the implementation provides a means to disable these
`runtime checks`{.interpreted-text role="idx"}. See the section
[pragmas](#pragmas) for details.

Whether a panic results in an exception or in a fatal error is
implementation specific. Thus the following program is invalid; even
though the code purports to catch the `IndexDefect` from an
out-of-bounds array access, the compiler may instead choose to allow the
program to die with a fatal error.

``` nim
var a: array[0..1, char]
let i = 5
try:
a[i] = 'N'
except IndexDefect:
echo "invalid index"
```

The current implementation allows to switch between these different
behaviors via `--panics:on|off`{.interpreted-text role="option"}. When
panics are turned on, the program dies with a panic, if they are turned
off the runtime errors are turned into exceptions. The benefit of
`--panics:on`{.interpreted-text role="option"} is that it produces
smaller binary code and the compiler has more freedom to optimize the
code.

An `unchecked runtime error`{.interpreted-text role="idx"} is an error
that is not guaranteed to be detected and can cause the subsequent
behavior of the computation to be arbitrary. Unchecked runtime errors
cannot occur if only `safe`{.interpreted-text role="idx"} language
features are used and if no runtime checks are disabled.

A `constant expression`{.interpreted-text role="idx"} is an expression
whose value can be computed during a semantic analysis of the code in
which it appears. It is never an l-value and never has side effects.
Constant expressions are not limited to the capabilities of semantic
analysis, such as constant folding; they can use all Nim language
features that are supported for compile-time execution. Since constant
expressions can be used as an input to semantic analysis (such as for
defining array bounds), this flexibility requires the compiler to
interleave semantic analysis and compile-time code execution.

It is mostly accurate to picture semantic analysis proceeding top to
bottom and left to right in the source code, with compile-time code
execution interleaved when necessary to compute values that are required
for subsequent semantic analysis. We will see much later in this
document that macro invocation not only requires this interleaving, but
also creates a situation where semantic analysis does not entirely
proceed top to bottom and left to right.