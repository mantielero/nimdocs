## Procedures

What most programming languages call `methods`{.interpreted-text
role="idx"} or `functions` are called
`procedures` in Nim. A procedure
declaration consists of an identifier, zero or more formal parameters, a
return value type and a block of code. Formal parameters are declared as
a list of identifiers separated by either comma or semicolon. A
parameter is given a type by `: typename`. The type applies to all
parameters immediately before it, until either the beginning of the
parameter list, a semicolon separator, or an already typed parameter, is
reached. The semicolon can be used to make separation of types and
subsequent identifiers more distinct.

``` nim
# Using only commas
proc foo(a, b: int, c, d: bool): int
# Using semicolon for visual distinction
proc foo(a, b: int; c, d: bool): int

# Will fail: a is untyped since ';' stops type propagation.
proc foo(a; b: int; c, d: bool): int
```

A parameter may be declared with a default value which is used if the
caller does not provide a value for the argument. The value will be
reevaluated every time the function is called.

``` nim
# b is optional with 47 as its default value
proc foo(a: int, b: int = 47): int
```

Parameters can be declared mutable and so allow the proc to modify those
arguments, by using the type modifier `var`.

``` nim
# "returning" a value to the caller through the 2nd argument
# Notice that the function uses no actual return value at all (ie void)
proc foo(inp: int, outp: var int) =
outp = inp + 47
```

If the proc declaration has no body, it is a `forward`{.interpreted-text
role="idx"} declaration. If the proc returns a value, the procedure body
can access an implicitly declared variable named
`result` that represents the return value.
Procs can be overloaded. The overloading resolution algorithm determines
which proc is the best match for the arguments. Example:

``` nim
proc toLower(c: char): char = # toLower for characters
  if c in {'A'..'Z'}:
    result = chr(ord(c) + (ord('a') - ord('A')))
  else:
    result = c

proc toLower(s: string): string = # toLower for strings
  result = newString(len(s))
  for i in 0..len(s) - 1:
    result[i] = toLower(s[i]) # calls toLower for characters; no recursion!
```

Calling a procedure can be done in many different ways:

``` nim
proc callme(x, y: int, s: string = "", c: char, b: bool = false) = ...
# call with positional arguments      # parameter bindings:
callme(0, 1, "abc", '\t', true)       # (x=0, y=1, s="abc", c='\t', b=true)
# call with named and positional arguments:
callme(y=1, x=0, "abd", '\t')         # (x=0, y=1, s="abd", c='\t', b=false)
# call with named arguments (order is not relevant):
callme(c='\t', y=1, x=0)              # (x=0, y=1, s="", c='\t', b=false)
# call as a command statement: no () needed:
callme 0, 1, "abc", '\t'              # (x=0, y=1, s="abc", c='\t', b=false)
```

A procedure may call itself recursively.

`Operators` are procedures with a special
operator symbol as identifier:

``` nim
proc `$` (x: int): string =
# converts an integer to a string; this is a prefix operator.
result = intToStr(x)
```

Operators with one parameter are prefix operators, operators with two
parameters are infix operators. (However, the parser distinguishes these
from the operator\'s position within an expression.) There is no way to
declare postfix operators: all postfix operators are built-in and
handled by the grammar explicitly.

Any operator can be called like an ordinary proc with the \`opr\`
notation. (Thus an operator can have more than two parameters):

``` nim
proc `*+` (a, b, c: int): int =
# Multiply and add
result = a * b + c
assert `*+`(3, 4, 6) == `+`(`*`(a, b), c)
```

### Export marker

If a declared symbol is marked with an `asterisk`{.interpreted-text
role="idx"} it is exported from the current module:

``` nim
proc exportedEcho*(s: string) = echo s
proc `*`*(a: string; b: int): string =
  result = newStringOfCap(a.len * b)
  for i in 1..b: result.add a

var exportedVar*: int
const exportedConst* = 78
type
  ExportedType* = object
    exportedField*: int
```

### Method call syntax

For object-oriented programming, the syntax `obj.methodName(args)` can
be used instead of `methodName(obj, args)`. The parentheses can be
omitted if there are no remaining arguments: `obj.len` (instead of
`len(obj)`).

This method call syntax is not restricted to objects, it can be used to
supply any type of first argument for procedures:

``` nim
echo "abc".len # is the same as echo len "abc"
echo "abc".toUpper()
echo {'a', 'b', 'c'}.card
stdout.writeLine("Hallo") # the same as writeLine(stdout, "Hallo")
```

Another way to look at the method call syntax is that it provides the
missing postfix notation.

The method call syntax conflicts with explicit generic instantiations:
`p[T](x)` cannot be written as `x.p[T]` because `x.p[T]` is always
parsed as `(x.p)[T]`.

See also: [Limitations of the method call
syntax](#templates-limitations-of-the-method-call-syntax).

The `[: ]` notation has been designed to mitigate this issue: `x.p[:T]`
is rewritten by the parser to `p[T](x)`, `x.p[:T](y)` is rewritten to
`p[T](x, y)`. Note that `[: ]` has no AST representation, the rewrite is
performed directly in the parsing step.

### Properties

Nim has no need for *get-properties*: Ordinary get-procedures that are
called with the *method call syntax* achieve the same. But setting a
value is different; for this, a special setter syntax is needed:

``` nim
# Module asocket
type
Socket* = ref object of RootObj
host: int # cannot be accessed from the outside of the module
proc `host=`*(s: var Socket, value: int) {.inline.} =
  ## setter of hostAddr.
  ## This accesses the 'host' field and is not a recursive call to
  ## `host=` because the builtin dot access is preferred if it is
  ## available:
  s.host = value

proc host*(s: Socket): int {.inline.} =
  ## getter of hostAddr
  ## This accesses the 'host' field and is not a recursive call to
  ## `host` because the builtin dot access is preferred if it is
  ## available:
  s.host
```

``` nim
# module B
import asocket
var s: Socket
new s
s.host = 34  # same as `host=`(s, 34)
```

A proc defined as `f=` (with the trailing `=`) is called a
`setter`. A setter can be called
explicitly via the common backticks notation:

``` nim
proc `f=`(x: MyObject; value: string) =
  discard

`f=`(myObject, "value")
```

`f=` can be called implicitly in the pattern `x.f = value` if and only
if the type of `x` does not have a field named `f` or if `f` is not
visible in the current module. These rules ensure that object fields and
accessors can have the same name. Within the module `x.f` is then always
interpreted as field access and outside the module it is interpreted as
an accessor proc call.

### Command invocation syntax

Routines can be invoked without the `()` if the call is syntactically a
statement. This command invocation syntax also works for expressions,
but then only a single argument may follow. This restriction means
`echo f 1, f 2` is parsed as `echo(f(1), f(2))` and not as
`echo(f(1, f(2)))`. The method call syntax may be used to provide one
more argument in this case:

``` nim
proc optarg(x: int, y: int = 0): int = x + y
proc singlearg(x: int): int = 20*x
echo optarg 1, " ", singlearg 2  # prints "1 40"

let fail = optarg 1, optarg 8   # Wrong. Too many arguments for a command call
let x = optarg(1, optarg 8)  # traditional procedure call with 2 arguments
let y = 1.optarg optarg 8    # same thing as above, w/o the parenthesis
assert x == y
```

The command invocation syntax also can\'t have complex expressions as
arguments. For example: ([anonymous
procs](#procedures-anonymous-procs)), `if`, `case` or `try`. Function
calls with no arguments still need () to distinguish between a call and
the function itself as a first-class value.

### Closures

Procedures can appear at the top level in a module as well as inside
other scopes, in which case they are called nested procs. A nested proc
can access local variables from its enclosing scope and if it does so it
becomes a closure. Any captured variables are stored in a hidden
additional argument to the closure (its environment) and they are
accessed by reference by both the closure and its enclosing scope (i.e.
any modifications made to them are visible in both places). The closure
environment may be allocated on the heap or on the stack if the compiler
determines that this would be safe.

#### Creating closures in loops

Since closures capture local variables by reference it is often not
wanted behavior inside loop bodies. See
[closureScope](system.html#closureScope.t,untyped) and
[capture](sugar.html#capture.m,varargs%5Btyped%5D,untyped) for details
on how to change this behavior.

### Anonymous Procs

Unnamed procedures can be used as lambda expressions to pass into other
procedures:

``` nim
var cities = @["Frankfurt", "Tokyo", "New York", "Kyiv"]
cities.sort(proc (x,y: string): int =
    cmp(x.len, y.len))
```

Procs as expressions can appear both as nested procs and inside
top-level executable code. The [sugar](sugar.html) module contains the
`=>` macro which enables a more succinct syntax for anonymous procedures
resembling lambdas as they are in languages like JavaScript, C#, etc.

### Func

The `func` keyword introduces a shortcut for a
`noSideEffect` proc.

``` nim
func binarySearch[T](a: openArray[T]; elem: T): int
```

Is short for:

``` nim
proc binarySearch[T](a: openArray[T]; elem: T): int {.noSideEffect.}
```

### Routines

A routine is a symbol of kind: `proc`, `func`, `method`, `iterator`,
`macro`, `template`, `converter`.

### Type bound operators

A type bound operator is a `proc` or `func` whose name starts with `=`
but isn\'t an operator (i.e. containing only symbols, such as `==`).
These are unrelated to setters (see
[properties](manual.html#procedures-properties)), which instead end in
`=`. A type bound operator declared for a type applies to the type
regardless of whether the operator is in scope (including if it is
private).

``` nim
# foo.nim:
var witness* = 0
type Foo[T] = object
proc initFoo*(T: typedesc): Foo[T] = discard
proc `=destroy`[T](x: var Foo[T]) = witness.inc # type bound operator
# main.nim:
import foo
block:
  var a = initFoo(int)
  doAssert witness == 0
doAssert witness == 1
block:
  var a = initFoo(int)
  doAssert witness == 1
  `=destroy`(a) # can be called explicitly, even without being in scope
  doAssert witness == 2
# will still be called upon exiting scope
doAssert witness == 3
```

Type bound operators are: `=destroy`, `=copy`, `=sink`, `=trace`,
`=deepcopy`.

For more details on some of those procs, see [Lifetime-tracking
hooks](destructors.html#lifetimeminustracking-hooks).

### Nonoverloadable builtins

The following built-in procs cannot be overloaded for reasons of
implementation simplicity (they require specialized semantic checking):

    declared, defined, definedInScope, compiles, sizeof,
    is, shallowCopy, getAst, astToStr, spawn, procCall

Thus they act more like keywords than like ordinary identifiers; unlike
a keyword however, a redefinition may `shadow`{.interpreted-text
role="idx"} the definition in the [system]() module. From this list the
following should not be written in dot notation `x.f` since `x` cannot
be type-checked before it gets passed to \`f\`:

    declared, defined, definedInScope, compiles, getAst, astToStr

### Var parameters

The type of a parameter may be prefixed with the `var` keyword:

``` nim
proc divmod(a, b: int; res, remainder: var int) =
res = a div b
remainder = a mod b
var
  x, y: int

divmod(8, 5, x, y) # modifies x and y
assert x == 1
assert y == 3
```

In the example, `res` and `remainder` are `var parameters`. Var
parameters can be modified by the procedure and the changes are visible
to the caller. The argument passed to a var parameter has to be an
l-value. Var parameters are implemented as hidden pointers. The above
example is equivalent to:

``` nim
proc divmod(a, b: int; res, remainder: ptr int) =
res[] = a div b
remainder[] = a mod b
var
  x, y: int
divmod(8, 5, addr(x), addr(y))
assert x == 1
assert y == 3
```

In the examples, var parameters or pointers are used to provide two
return values. This can be done in a cleaner way by returning a tuple:

``` nim
proc divmod(a, b: int): tuple[res, remainder: int] =
(a div b, a mod b)
var t = divmod(8, 5)

assert t.res == 1
assert t.remainder == 3
```

One can use `tuple unpacking` to access
the tuple\'s fields:

``` nim
var (x, y) = divmod(8, 5) # tuple unpacking
assert x == 1
assert y == 3
```

**Note**: `var` parameters are never necessary for efficient parameter
passing. Since non-var parameters cannot be modified the compiler is
always free to pass arguments by reference if it considers it can speed
up execution.

### Var return type

A proc, converter, or iterator may return a `var` type which means that
the returned value is an l-value and can be modified by the caller:

``` nim
var g = 0
proc writeAccessToG(): var int =
  result = g

writeAccessToG() = 6
assert g == 6
```

It is a static error if the implicitly introduced pointer could be used
to access a location beyond its lifetime:

``` nim
proc writeAccessToG(): var int =
var g = 0
result = g # Error!
```

For iterators, a component of a tuple return type can have a `var` type
too:

``` nim
iterator mpairs(a: var seq[string]): tuple[key: int, val: var string] =
for i in 0..a.high:
yield (i, a[i])
```

In the standard library every name of a routine that returns a `var`
type starts with the prefix `m` per convention.

#### Future directions

Later versions of Nim can be more precise about the borrowing rule with
a syntax like:

``` nim
proc foo(other: Y; container: var X): var T from container
```

Here `var T from container` explicitly exposes that the location is
derived from the second parameter (called \'container\' in this case).
The syntax `var T from p` specifies a type `varTy[T, 2]` which is
incompatible with `varTy[T, 1]`.

### NRVO

**Note**: This section describes the current implementation. This part
of the language specification will be changed. See
<https://github.com/nim-lang/RFCs/issues/230> for more information.

The return value is represented inside the body of a routine as the
special `result` variable. This allows for
a mechanism much like C++\'s \"named return value optimization\"
(`NRVO`). NRVO means that the stores to
`result` inside `p` directly affect the destination `dest` in
`let/var dest = p(args)` (definition of `dest`) and also in
`dest = p(args)` (assignment to `dest`). This is achieved by rewriting
`dest = p(args)` to `p'(args, dest)` where `p'` is a variation of `p`
that returns `void` and receives a hidden mutable parameter representing
`result`.

Informally:

``` nim
proc p(): BigT = ...
var x = p()
x = p()

# is roughly turned into:

proc p(result: var BigT) = ...

var x; p(x)
p(x)
```

Let `T`\'s be `p`\'s return type. NRVO applies for `T` if
`sizeof(T) >= N` (where `N` is implementation dependent), in other
words, it applies for \"big\" structures.

If `p` can raise an exception, NRVO applies regardless. This can produce
observable differences in behavior:

``` nim
type
  BigT = array[16, int]

proc p(raiseAt: int): BigT =
  for i in 0..high(result):
    if i == raiseAt: raise newException(ValueError, "interception")
    result[i] = i

proc main =
  var x: BigT
  try:
    x = p(8)
  except ValueError:
    doAssert x == [0, 1, 2, 3, 4, 5, 6, 7, 0, 0, 0, 0, 0, 0, 0, 0]

main()
```

However, the current implementation produces a warning in these cases.
There are different ways to deal with this warning:

1.  Disable the warning via `{.push warning[ObservableStores]: off.}`
    \... `{.pop.}`. Then one may need to ensure that `p` only raises
    *before* any stores to `result` happen.
2.  One can use a temporary helper variable, for example instead of
    `x = p(8)` use `let tmp = p(8); x = tmp`.

### Overloading of the subscript operator

The `[]` subscript operator for arrays/openarrays/sequences can be
overloaded.
