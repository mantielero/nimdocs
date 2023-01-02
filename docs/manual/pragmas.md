## Pragmas

Pragmas are Nim\'s method to give the compiler additional information /
commands without introducing a massive number of new keywords. Pragmas
are processed on the fly during semantic checking. Pragmas are enclosed
in the special `{.` and `.}` curly brackets. Pragmas are also often used
as a first implementation to play with a language feature before a nicer
syntax to access the feature becomes available.

### deprecated pragma

The deprecated pragma is used to mark a symbol as deprecated:

``` nim
proc p() {.deprecated.}
var x {.deprecated.}: char
```

This pragma can also take in an optional warning string to relay to
developers.

``` nim
proc thing(x: bool) {.deprecated: "use thong instead".}
```

### compileTime pragma

The `compileTime` pragma is used to mark a proc or variable to be used
only during compile-time execution. No code will be generated for it.
Compile-time procs are useful as helpers for macros. Since version
0.12.0 of the language, a proc that uses `system.NimNode` within its
parameter types is implicitly declared \`compileTime\`:

``` nim
proc astHelper(n: NimNode): NimNode =
result = n
```

Is the same as:

``` nim
proc astHelper(n: NimNode): NimNode {.compileTime.} =
result = n
```

`compileTime` variables are available at runtime too. This simplifies
certain idioms where variables are filled at compile-time (for example,
lookup tables) but accessed at runtime:

``` {.nim test="\"nim c -r $1\""}
import std/macros

var nameToProc {.compileTime.}: seq[(string, proc (): string {.nimcall.})]

macro registerProc(p: untyped): untyped =
  result = newTree(nnkStmtList, p)

  let procName = p[0]
  let procNameAsStr = $p[0]
  result.add quote do:
    nameToProc.add((`procNameAsStr`, `procName`))

proc foo: string {.registerProc.} = "foo"
proc bar: string {.registerProc.} = "bar"
proc baz: string {.registerProc.} = "baz"

doAssert nameToProc[2][1]() == "baz"
```

### noReturn pragma

The `noreturn` pragma is used to mark a proc that never returns.

### acyclic pragma

The `acyclic` pragma can be used for object types to mark them as
acyclic even though they seem to be cyclic. This is an **optimization**
for the garbage collector to not consider objects of this type as part
of a cycle:

``` nim
type
Node = ref NodeObj
NodeObj {.acyclic.} = object
left, right: Node
data: string
```

Or if we directly use a ref object:

``` nim
type
Node {.acyclic.} = ref object
left, right: Node
data: string
```

In the example, a tree structure is declared with the `Node` type. Note
that the type definition is recursive and the GC has to assume that
objects of this type may form a cyclic graph. The `acyclic` pragma
passes the information that this cannot happen to the GC. If the
programmer uses the `acyclic` pragma for data types that are in reality
cyclic, this may result in memory leaks, but memory safety is preserved.

### final pragma

The `final` pragma can be used for an object type to specify that it
cannot be inherited from. Note that inheritance is only available for
objects that inherit from an existing object (via the
`object of SuperType` syntax) or that have been marked as `inheritable`.

### shallow pragma

The `shallow` pragma affects the semantics of a type: The compiler is
allowed to make a shallow copy. This can cause serious semantic issues
and break memory safety! However, it can speed up assignments
considerably, because the semantics of Nim require deep copying of
sequences and strings. This can be expensive, especially if sequences
are used to build a tree structure:

``` nim
type
NodeKind = enum nkLeaf, nkInner
Node {.shallow.} = object
case kind: NodeKind
of nkLeaf:
strVal: string
of nkInner:
children: seq[Node]
```

### pure pragma

An object type can be marked with the `pure` pragma so that its type
field which is used for runtime type identification is omitted. This
used to be necessary for binary compatibility with other compiled
languages.

An enum type can be marked as `pure`. Then access of its fields always
requires full qualification.

### asmNoStackFrame pragma

A proc can be marked with the `asmNoStackFrame` pragma to tell the
compiler it should not generate a stack frame for the proc. There are
also no exit statements like `return result;` generated and the
generated C function is declared as
`__declspec(naked)`{.interpreted-text role="c"} or
`__attribute__((naked))`{.interpreted-text role="c"} (depending on the
used C compiler).

**Note**: This pragma should only be used by procs which consist solely
of assembler statements.

### error pragma

The `error` pragma is used to make the compiler output an error message
with the given content. The compilation does not necessarily abort after
an error though.

The `error` pragma can also be used to annotate a symbol (like an
iterator or proc). The *usage* of the symbol then triggers a static
error. This is especially useful to rule out that some operation is
valid due to overloading and type conversions:

``` nim
## check that underlying int values are compared and not the pointers:
proc `==`(x, y: ptr int): bool {.error.}
```

### fatal pragma

The `fatal` pragma is used to make the compiler output an error message
with the given content. In contrast to the `error` pragma, the
compilation is guaranteed to be aborted by this pragma. Example:

``` nim
when not defined(objc):
{.fatal: "Compile this program with the objc command!".}
```

### warning pragma

The `warning` pragma is used to make the compiler output a warning
message with the given content. Compilation continues after the warning.

### hint pragma

The `hint` pragma is used to make the compiler output a hint message
with the given content. Compilation continues after the hint.

### line pragma

The `line` pragma can be used to affect line information of the
annotated statement, as seen in stack backtraces:

``` nim
template myassert*(cond: untyped, msg = "") =
  if not cond:
    # change run-time line information of the 'raise' statement:
    {.line: instantiationInfo().}:
      raise newException(AssertionDefect, msg)
```

If the `line` pragma is used with a parameter, the parameter needs be a
`tuple[filename: string, line: int]`. If it is used without a parameter,
`system.instantiationInfo()` is used.

### linearScanEnd pragma

The `linearScanEnd` pragma can be used to tell the compiler how to
compile a Nim `case`{.interpreted-text role="idx"} statement.
Syntactically it has to be used as a statement:

``` nim
case myInt
of 0:
echo "most common case"
of 1:
{.linearScanEnd.}
echo "second most common case"
of 2: echo "unlikely: use branch table"
else: echo "unlikely too: use branch table for ", myInt
```

In the example, the case branches `0` and `1` are much more common than
the other cases. Therefore the generated assembler code should test for
these values first so that the CPU\'s branch predictor has a good chance
to succeed (avoiding an expensive CPU pipeline stall). The other cases
might be put into a jump table for O(1) overhead but at the cost of a
(very likely) pipeline stall.

The `linearScanEnd` pragma should be put into the last branch that
should be tested against via linear scanning. If put into the last
branch of the whole `case` statement, the whole `case` statement uses
linear scanning.

### computedGoto pragma

The `computedGoto` pragma can be used to tell the compiler how to
compile a Nim `case`{.interpreted-text role="idx"} in a `while true`
statement. Syntactically it has to be used as a statement inside the
loop:

``` nim
type
  MyEnum = enum
    enumA, enumB, enumC, enumD, enumE

proc vm() =
  var instructions: array[0..100, MyEnum]
  instructions[2] = enumC
  instructions[3] = enumD
  instructions[4] = enumA
  instructions[5] = enumD
  instructions[6] = enumC
  instructions[7] = enumA
  instructions[8] = enumB

  instructions[12] = enumE
  var pc = 0
  while true:
    {.computedGoto.}
    let instr = instructions[pc]
    case instr
    of enumA:
      echo "yeah A"
    of enumC, enumD:
      echo "yeah CD"
    of enumB:
      echo "yeah B"
    of enumE:
      break
    inc(pc)

vm()
```

As the example shows, `computedGoto` is mostly useful for interpreters.
If the underlying backend (C compiler) does not support the computed
goto extension the pragma is simply ignored.

### immediate pragma

The immediate pragma is obsolete. See [Typed vs untyped
parameters](#templates-typed-vs-untyped-parameters).

### compilation option pragmas

The listed pragmas here can be used to override the code generation
options for a proc/method/converter.

The implementation currently provides the following possible options
(various others may be added later).

  pragma           allowed values   description
  ---------------- ---------------- ------------------------------------------------------------------------------------------------
  checks           on\|off          Turns the code generation for all runtime checks on or off.
  boundChecks      on\|off          Turns the code generation for array bound checks on or off.
  overflowChecks   on\|off          Turns the code generation for over- or underflow checks on or off.
  nilChecks        on\|off          Turns the code generation for nil pointer checks on or off.
  assertions       on\|off          Turns the code generation for assertions on or off.
  warnings         on\|off          Turns the warning messages of the compiler on or off.
  hints            on\|off          Turns the hint messages of the compiler on or off.
  optimization     nonesize         Optimize the code for speed or size, or disable optimization.
  patterns         on\|off          Turns the term rewriting templates/macros on or off.
  callconv         cdecl\|\...      Specifies the default calling convention for all procedures (and procedure types) that follow.

Example:

``` nim
{.checks: off, optimization: speed.}
# compile without runtime checks and optimize for speed
```

### push and pop pragmas

The `push/pop`{.interpreted-text role="idx"} pragmas are very similar to
the option directive, but are used to override the settings temporarily.
Example:

``` nim
{.push checks: off.}
# compile this section without runtime checks as it is
# speed critical
# ... some code ...
{.pop.} # restore old settings
```

`push/pop`{.interpreted-text role="idx"} can switch on/off some standard
library pragmas, example:

``` nim
{.push inline.}
proc thisIsInlined(): int = 42
func willBeInlined(): float = 42.0
{.pop.}
proc notInlined(): int = 9
{.push discardable, boundChecks: off, compileTime, noSideEffect, experimental.}
template example(): string = "https://nim-lang.org"
{.pop.}

{.push deprecated, hint[LineTooLong]: off, used, stackTrace: off.}
proc sample(): bool = true
{.pop.}
```

For third party pragmas, it depends on its implementation but uses the
same syntax.

### register pragma

The `register` pragma is for variables only. It declares the variable as
`register`, giving the compiler a hint that the variable should be
placed in a hardware register for faster access. C compilers usually
ignore this though and for good reasons: Often they do a better job
without it anyway.

However, in highly specific cases (a dispatch loop of a bytecode
interpreter for example) it may provide benefits.

### global pragma

The `global` pragma can be applied to a variable within a proc to
instruct the compiler to store it in a global location and initialize it
once at program startup.

``` nim
proc isHexNumber(s: string): bool =
var pattern {.global.} = re"[0-9a-fA-F]+"
result = s.match(pattern)
```

When used within a generic proc, a separate unique global variable will
be created for each instantiation of the proc. The order of
initialization of the created global variables within a module is not
defined, but all of them will be initialized after any top-level
variables in their originating module and before any variable in a
module that imports it.

### Disabling certain messages

Nim generates some warnings and hints (\"line too long\") that may annoy
the user. A mechanism for disabling certain messages is provided: Each
hint and warning message contains a symbol in brackets. This is the
message\'s identifier that can be used to enable or disable it:

``` Nim
{.hint[LineTooLong]: off.} # turn off the hint about too long lines
```

This is often better than disabling all warnings at once.

### used pragma

Nim produces a warning for symbols that are not exported and not used
either. The `used` pragma can be attached to a symbol to suppress this
warning. This is particularly useful when the symbol was generated by a
macro:

``` nim
template implementArithOps(T) =
proc echoAdd(a, b: T) {.used.} =
echo a + b
proc echoSub(a, b: T) {.used.} =
echo a - b
# no warning produced for the unused 'echoSub'
implementArithOps(int)
echoAdd 3, 5
```

`used` can also be used as a top-level statement to mark a module as
\"used\". This prevents the \"Unused import\" warning:

``` nim
# module: debughelper.nim
when defined(nimHasUsed):
  # 'import debughelper' is so useful for debugging
  # that Nim shouldn't produce a warning for that import,
  # even if currently unused:
  {.used.}
```

### experimental pragma

The `experimental` pragma enables experimental language features.
Depending on the concrete feature, this means that the feature is either
considered too unstable for an otherwise stable release or that the
future of the feature is uncertain (it may be removed at any time).

Example:

``` nim
import std/threadpool
{.experimental: "parallel".}
proc threadedEcho(s: string, i: int) =
  echo(s, " ", $i)

proc useParallel() =
  parallel:
    for i in 0..4:
      spawn threadedEcho("echo in parallel", i)

useParallel()
```

As a top-level statement, the experimental pragma enables a feature for
the rest of the module it\'s enabled in. This is problematic for macro
and generic instantiations that cross a module scope. Currently, these
usages have to be put into a `.push/pop` environment:

``` nim
# client.nim
proc useParallel*[T](unused: T) =
  # use a generic T here to show the problem.
  {.push experimental: "parallel".}
  parallel:
    for i in 0..4:
      echo "echo in parallel"

  {.pop.}
```

``` nim
import client
useParallel(1)
```