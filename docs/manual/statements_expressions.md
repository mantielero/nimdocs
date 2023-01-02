## Statements and expressions

Nim uses the common statement/expression paradigm: Statements do not
produce a value in contrast to expressions. However, some expressions
are statements.

Statements are separated into `simple statements`{.interpreted-text
role="idx"} and `complex statements`.
Simple statements are statements that cannot contain other statements
like assignments, calls, or the `return` statement; complex statements
can contain other statements. To avoid the
`dangling else problem`, complex
statements always have to be indented. The details can be found in the
grammar.

### Statement list expression

Statements can also occur in an expression context that looks like
`(stmt1; stmt2; ...; ex)`. This is called a statement list expression or
`(;)`. The type of `(stmt1; stmt2; ...; ex)` is the type of `ex`. All
the other statements must be of type `void`. (One can use `discard` to
produce a `void` type.) `(;)` does not introduce a new scope.

### Discard statement

Example:

``` nim
proc p(x, y: int): int =
result = x + y
discard p(3, 4) # discard the return value of `p`
```

The `discard` statement evaluates its expression for side-effects and
throws the expression\'s resulting value away, and should only be used
when ignoring this value is known not to cause problems.

Ignoring the return value of a procedure without using a discard
statement is a static error.

The return value can be ignored implicitly if the called proc/iterator
has been declared with the `discardable`
pragma:

``` nim
proc p(x, y: int): int {.discardable.} =
result = x + y
p(3, 4) # now valid
```

however the discardable pragma does not work on templates as templates
substitute the AST in place. For example:

``` nim
{.push discardable .}
template example(): string = "https://nim-lang.org"
{.pop.}
example()
```

This template will resolve into \"<https://nim-lang.org>\" which is a
string literal and since {.discardable.} doesn\'t apply to literals, the
compiler will error.

An empty `discard` statement is often used as a null statement:

``` nim
proc classify(s: string) =
case s[0]
of SymChars, '_': echo "an identifier"
of '0'..'9': echo "a number"
else: discard
```

### Void context

In a list of statements, every expression except the last one needs to
have the type `void`. In addition to this rule an assignment to the
builtin `result` symbol also triggers a mandatory `void` context for the
subsequent expressions:

``` nim
proc invalid*(): string =
result = "foo"
"invalid"  # Error: value of type 'string' has to be discarded
```

``` nim
proc valid*(): string =
let x = 317
"valid"
```

### Var statement

Var statements declare new local and global variables and initialize
them. A comma-separated list of variables can be used to specify
variables of the same type:

``` nim
var
  a: int = 0
  x, y, z: int
```

If an initializer is given, the type can be omitted: the variable is
then of the same type as the initializing expression. Variables are
always initialized with a default value if there is no initializing
expression. The default value depends on the type and is always a zero
in binary.

  Type                        default value
  --------------------------- --------------------------------------------------------
  any integer type            0
  any float                   0.0
  char                        \'\\0\'
  bool                        false
  ref or pointer type         nil
  procedural type             nil
  sequence                    `@[]`
  string                      `""`
  tuple\[x: A, y: B, \...\]   (default(A), default(B), \...) (analogous for objects)
  array\[0\..., T\]           \[default(T), \...\]
  range\[T\]                  default(T); this may be out of the valid range
  T = enum                    cast\[T\](0); this may be an invalid value

The implicit initialization can be avoided for optimization reasons with
the `noinit` pragma:

``` nim
var
a {.noInit.}: array[0..1023, char]
```

If a proc is annotated with the `noinit` pragma, this refers to its
implicit `result` variable:

``` nim
proc returnUndefinedValue: int {.noinit.} = discard
```

The implicit initialization can also be prevented by the
`requiresInit` type pragma. The compiler
requires an explicit initialization for the object and all of its
fields. However, it does a `control flow analysis`{.interpreted-text
role="idx"} to prove the variable has been initialized and does not rely
on syntactic properties:

``` nim
type
MyObject = object {.requiresInit.}
proc p() =
  # the following is valid:
  var x: MyObject
  if someCondition():
    x = a()
  else:
    x = a()
  # use x
```

`requiresInit` pragma can also be applyied to `distinct` types.

Given the following distinct type definitions:

``` nim
type
DistinctObject {.requiresInit, borrow: `.`.} = distinct MyObject
DistinctString {.requiresInit.} = distinct string
```

The following code blocks will fail to compile:

``` nim
var foo: DistinctFoo
foo.x = "test"
doAssert foo.x == "test"
```

``` nim
var s: DistinctString
s = "test"
doAssert s == "test"
```

But these ones will compile successfully:

``` nim
let foo = DistinctFoo(Foo(x: "test"))
doAssert foo.x == "test"
```

``` nim
let s = "test"
doAssert s == "test"
```

### Let statement

A `let` statement declares new local and global
`single assignment` variables and binds a
value to them. The syntax is the same as that of the `var` statement,
except that the keyword `var` is replaced by the keyword `let`. Let
variables are not l-values and can thus not be passed to `var`
parameters nor can their address be taken. They cannot be assigned new
values.

For let variables, the same pragmas are available as for ordinary
variables.

As `let` statements are immutable after creation they need to define a
value when they are declared. The only exception to this is if the
`{.importc.}` pragma (or any of the other `importX` pragmas) is applied,
in this case the value is expected to come from native code, typically a
C/C++ `const`.

### Tuple unpacking

In a `var` or `let` statement tuple unpacking can be performed. The
special identifier `_` can be used to ignore some parts of the tuple:

``` nim
proc returnsTuple(): (int, int, int) = (4, 2, 3)
let (x, _, z) = returnsTuple()
```

### Const section

A const section declares constants whose values are constant
expressions:

``` import std/[strutils]
const
roundPi = 3.1415
constEval = contains("abc", 'b') # computed at compile time!
```

Once declared, a constant\'s symbol can be used as a constant
expression.

See [Constants and Constant
Expressions](#constants-and-constant-expressions) for details.

### Static statement/expression

A static statement/expression explicitly requires compile-time
execution. Even some code that has side effects is permitted in a static
block:

``` 
static:
  echo "echo at compile time"
```

There are limitations on what Nim code can be executed at compile time;
see [Restrictions on Compile-Time
Execution](#restrictions-on-compileminustime-execution) for details.
It\'s a static error if the compiler cannot execute the block at compile
time.

### If statement

Example:

``` nim
var name = readLine(stdin)

if name == "Andreas":
  echo "What a nice name!"
elif name == "":
  echo "Don't you have a name?"
else:
  echo "Boring name..."
```

The `if` statement is a simple way to make a branch in the control flow:
The expression after the keyword `if` is evaluated, if it is true the
corresponding statements after the `:` are executed. Otherwise the
expression after the `elif` is evaluated (if there is an `elif` branch),
if it is true the corresponding statements after the `:` are executed.
This goes on until the last `elif`. If all conditions fail, the `else`
part is executed. If there is no `else` part, execution continues with
the next statement.

In `if` statements, new scopes begin immediately after the
`if`/`elif`/`else` keywords and ends after the corresponding *then*
block. For visualization purposes the scopes have been enclosed in
`{|  |}` in the following example:

``` nim
if {| (let m = input =~ re"(\w+)=\w+"; m.isMatch):
echo "key ", m[0], " value ", m[1]  |}
elif {| (let m = input =~ re""; m.isMatch):
echo "new m in this scope"  |}
else: {|
echo "m not declared here"  |}
```

### Case statement

Example:

``` nim
let line = readline(stdin)
case line
of "delete-everything", "restart-computer":
  echo "permission denied"
of "go-for-a-walk":     echo "please yourself"
elif line.len == 0:     echo "empty" # optional, must come after `of` branches
else:                   echo "unknown command" # ditto

# indentation of the branches is also allowed; and so is an optional colon
# after the selecting expression:
case readline(stdin):
  of "delete-everything", "restart-computer":
    echo "permission denied"
  of "go-for-a-walk":     echo "please yourself"
  else:                   echo "unknown command"
```

The `case` statement is similar to the `if` statement, but it represents
a multi-branch selection. The expression after the keyword `case` is
evaluated and if its value is in a *slicelist* the corresponding
statements (after the `of` keyword) are executed. If the value is not in
any given *slicelist*, trailing `elif` and `else` parts are executed
using same semantics as for `if` statement, and `elif` is handled just
like `else: if`. If there are no `else` or `elif` parts and not all
possible values that `expr` can hold occur in a *slicelist*, a static
error occurs. This holds only for expressions of ordinal types. \"All
possible values\" of `expr` are determined by `expr`\'s type. To
suppress the static error an `else: discard` should be used.

For non-ordinal types, it is not possible to list every possible value
and so these always require an `else` part. An exception to this rule is
for the `string` type, which currently doesn\'t require a trailing
`else` or `elif` branch; it\'s unspecified whether this will keep
working in future versions.

Because case statements are checked for exhaustiveness during semantic
analysis, the value in every `of` branch must be a constant expression.
This restriction also allows the compiler to generate more performant
code.

As a special semantic extension, an expression in an `of` branch of a
case statement may evaluate to a set or array constructor; the set or
array is then expanded into a list of its elements:

``` nim
const
SymChars: set[char] = {'a'..'z', 'A'..'Z', '\x80'..'\xFF'}
proc classify(s: string) =
  case s[0]
  of SymChars, '_': echo "an identifier"
  of '0'..'9': echo "a number"
  else: echo "other"

# is equivalent to:
proc classify(s: string) =
  case s[0]
  of 'a'..'z', 'A'..'Z', '\x80'..'\xFF', '_': echo "an identifier"
  of '0'..'9': echo "a number"
  else: echo "other"
```

The `case` statement doesn\'t produce an l-value, so the following
example won\'t work:

``` nim
type
Foo = ref object
x: seq[string]
proc get_x(x: Foo): var seq[string] =
  # doesn't work
  case true
  of true:
    x.x
  else:
    x.x

var foo = Foo(x: @[])
foo.get_x().add("asd")
```

This can be fixed by explicitly using `result` or \`return\`:

``` nim
proc get_x(x: Foo): var seq[string] =
case true
of true:
result = x.x
else:
result = x.x
```

### When statement

Example:

``` nim
when sizeof(int) == 2:
  echo "running on a 16 bit system!"
elif sizeof(int) == 4:
  echo "running on a 32 bit system!"
elif sizeof(int) == 8:
  echo "running on a 64 bit system!"
else:
  echo "cannot happen!"
```

The `when` statement is almost identical to the `if` statement with some
exceptions:

-   Each condition (`expr`) has to be a constant expression (of type
    `bool`).
-   The statements do not open a new scope.
-   The statements that belong to the expression that evaluated to true
    are translated by the compiler, the other statements are not checked
    for semantics! However, each condition is checked for semantics.

The `when` statement enables conditional compilation techniques. As a
special syntactic extension, the `when` construct is also available
within `object` definitions.

### When nimvm statement

`nimvm` is a special symbol that may be used as the expression of a
`when nimvm` statement to differentiate the execution path between
compile-time and the executable.

Example:

``` nim
proc someProcThatMayRunInCompileTime(): bool =
when nimvm:
# This branch is taken at compile time.
result = true
else:
# This branch is taken in the executable.
result = false
const ctValue = someProcThatMayRunInCompileTime()
let rtValue = someProcThatMayRunInCompileTime()
assert(ctValue == true)
assert(rtValue == false)
```

A `when nimvm` statement must meet the following requirements:

-   Its expression must always be `nimvm`. More complex expressions are
    not allowed.
-   It must not contain `elif` branches.
-   It must contain an `else` branch.
-   Code in branches must not affect semantics of the code that follows
    the `when nimvm` statement. E.g. it must not define symbols that are
    used in the following code.

### Return statement

Example:

``` nim
return 40+2
```

The `return` statement ends the execution of the current procedure. It
is only allowed in procedures. If there is an `expr`, this is syntactic
sugar for:

``` nim
result = expr
return result
```

`return` without an expression is a short notation for `return result`
if the proc has a return type. The `result`{.interpreted-text
role="idx"} variable is always the return value of the procedure. It is
automatically declared by the compiler. As all variables, `result` is
initialized to (binary) zero:

``` nim
proc returnZero(): int =
# implicitly returns 0
```

### Yield statement

Example:

``` nim
yield (1, 2, 3)
```

The `yield` statement is used instead of the `return` statement in
iterators. It is only valid in iterators. Execution is returned to the
body of the for loop that called the iterator. Yield does not end the
iteration process, but the execution is passed back to the iterator if
the next iteration starts. See the section about iterators ([Iterators
and the for statement](#iterators-and-the-for-statement)) for further
information.

### Block statement

Example:

``` nim
var found = false
block myblock:
for i in 0..3:
for j in 0..3:
if a[j][i] == 7:
found = true
break myblock # leave the block, in this case both for-loops
echo found
```

The block statement is a means to group statements to a (named) `block`.
Inside the block, the `break` statement is allowed to leave the block
immediately. A `break` statement can contain a name of a surrounding
block to specify which block is to be left.

### Break statement

Example:

``` nim
break
```

The `break` statement is used to leave a block immediately. If `symbol`
is given, it is the name of the enclosing block that is to be left. If
it is absent, the innermost block is left.

### While statement

Example:

``` nim
echo "Please tell me your password:"
var pw = readLine(stdin)
while pw != "12345":
echo "Wrong password! Next try:"
pw = readLine(stdin)
```

The `while` statement is executed until the `expr` evaluates to false.
Endless loops are no error. `while` statements open an `implicit block`
so that they can be left with a `break` statement.

### Continue statement

A `continue` statement leads to the immediate next iteration of the
surrounding loop construct. It is only allowed within a loop. A continue
statement is syntactic sugar for a nested block:

``` nim
while expr1:
stmt1
continue
stmt2
```

Is equivalent to:

``` nim
while expr1:
block myBlockName:
stmt1
break myBlockName
stmt2
```

### Assembler statement

The direct embedding of assembler code into Nim code is supported by the
unsafe `asm` statement. Identifiers in the assembler code that refer to
Nim identifiers shall be enclosed in a special character which can be
specified in the statement\'s pragmas. The default special character is
\`\'\`\'\`:

``` nim
{.push stackTrace:off.}
proc addInt(a, b: int): int =
# a in eax, and b in edx
asm """
mov eax, `a`
add eax, `b`
jno theEnd
call `raiseOverflow`
theEnd:
"""
{.pop.}
```

If the GNU assembler is used, quotes and newlines are inserted
automatically:

``` nim
proc addInt(a, b: int): int =
asm """
addl %%ecx, %%eax
jno 1
call `raiseOverflow`
1:
:"=a"(`result`)
:"a"(`a`), "c"(`b`)
"""
```

Instead of:

``` nim
proc addInt(a, b: int): int =
asm """
"addl %%ecx, %%eax\n"
"jno 1\n"
"call `raiseOverflow`\n"
"1: \n"
:"=a"(`result`)
:"a"(`a`), "c"(`b`)
"""
```

### Using statement

The `using` statement provides syntactic convenience in modules where
the same parameter names and types are used over and over. Instead of:

``` nim
proc foo(c: Context; n: Node) = ...
proc bar(c: Context; n: Node, counter: int) = ...
proc baz(c: Context; n: Node) = ...
```

One can tell the compiler about the convention that a parameter of name
`c` should default to type `Context`, `n` should default to `Node` etc.:

``` nim
using
c: Context
n: Node
counter: int
proc foo(c, n) = ...
proc bar(c, n, counter) = ...
proc baz(c, n) = ...

proc mixedMode(c, n; x, y: int) =
  # 'c' is inferred to be of the type 'Context'
  # 'n' is inferred to be of the type 'Node'
  # But 'x' and 'y' are of type 'int'.
```

The `using` section uses the same indentation based grouping syntax as a
`var` or `let` section.

Note that `using` is not applied for `template` since the untyped
template parameters default to the type `system.untyped`.

Mixing parameters that should use the `using` declaration with
parameters that are explicitly typed is possible and requires a
semicolon between them.

### If expression

An `if` expression is almost like an if statement, but it is an
expression. This feature is similar to *ternary operators* in other
languages. Example:

``` nim
var y = if x > 8: 9 else: 10
```

An if expression always results in a value, so the `else` part is
required. `Elif` parts are also allowed.

### When expression

Just like an `if` expression, but corresponding to the `when` statement.

### Case expression

The `case` expression is again very similar to the case statement:

``` nim
var favoriteFood = case animal
of "dog": "bones"
of "cat": "mice"
elif animal.endsWith"whale": "plankton"
else:
echo "I'm not sure what to serve, but everybody loves ice cream"
"ice cream"
```

As seen in the above example, the case expression can also introduce
side effects. When multiple statements are given for a branch, Nim will
use the last expression as the result value.

### Block expression

A `block` expression is almost like a block statement, but it is an
expression that uses the last expression under the block as the value.
It is similar to the statement list expression, but the statement list
expression does not open a new block scope.

``` nim
let a = block:
var fib = @[0, 1]
for i in 0..10:
fib.add fib[^1] + fib[^2]
fib
```

### Table constructor

A table constructor is syntactic sugar for an array constructor:

``` nim
{"key1": "value1", "key2", "key3": "value2"}
# is the same as:
[("key1", "value1"), ("key2", "value2"), ("key3", "value2")]
```

The empty table can be written `{:}` (in contrast to the empty set which
is `{}`) which is thus another way to write the empty array constructor
`[]`. This slightly unusual way of supporting tables has lots of
advantages:

-   The order of the (key,value)-pairs is preserved, thus it is easy to
    support ordered dicts with for example `{key: val}.newOrderedTable`.
-   A table literal can be put into a `const` section and the compiler
    can easily put it into the executable\'s data section just like it
    can for arrays and the generated data section requires a minimal
    amount of memory.
-   Every table implementation is treated equally syntactically.
-   Apart from the minimal syntactic sugar, the language core does not
    need to know about tables.

### Type conversions

Syntactically a *type conversion* is like a procedure call, but a type
name replaces the procedure name. A type conversion is always safe in
the sense that a failure to convert a type to another results in an
exception (if it cannot be determined statically).

Ordinary procs are often preferred over type conversions in Nim: For
instance, `$` is the `toString` operator by convention and `toFloat` and
`toInt` can be used to convert from floating-point to integer or vice
versa.

Type conversion can also be used to disambiguate overloaded routines:

``` nim
proc p(x: int) = echo "int"
proc p(x: string) = echo "string"

let procVar = (proc(x: string))(p)
procVar("a")
```

Since operations on unsigned numbers wrap around and are unchecked so
are type conversions to unsigned integers and between unsigned integers.
The rationale for this is mostly better interoperability with the C
Programming language when algorithms are ported from C to Nim.

Exception: Values that are converted to an unsigned type at compile time
are checked so that code like `byte(-1)` does not compile.

**Note**: Historically the operations were unchecked and the conversions
were sometimes checked but starting with the revision 1.0.4 of this
document and the language implementation the conversions too are now
*always unchecked*.

### Type casts

*Type casts* are a crude mechanism to interpret the bit pattern of an
expression as if it would be of another type. Type casts are only needed
for low-level programming and are inherently unsafe.

``` nim
cast[int](x)
```

The target type of a cast must be a concrete type, for instance, a
target type that is a type class (which is non-concrete) would be
invalid:

``` nim
type Foo = int or float
var x = cast[Foo](1) # Error: cannot cast to a non concrete type: 'Foo'
```

Type casts should not be confused with *type conversions,* as mentioned
in the prior section. Unlike type conversions, a type cast cannot change
the underlying bit pattern of the data being casted (aside from that the
size of the target type may differ from the source type). Casting
resembles *type punning* in other languages or C++\'s
`reinterpret_cast`{.interpreted-text role="cpp"} and
`bit_cast`{.interpreted-text role="cpp"} features.

### The addr operator

The `addr` operator returns the address of an l-value. If the type of
the location is `T`, the `addr` operator result is of the type `ptr T`.
An address is always an untraced reference. Taking the address of an
object that resides on the stack is **unsafe**, as the pointer may live
longer than the object on the stack and can thus reference a
non-existing object. One can get the address of variables, but one
can\'t use it on variables declared through `let` statements:

``` nim
let t1 = "Hello"
var
  t2 = t1
  t3 : pointer = addr(t2)
echo repr(addr(t2))
# --> ref 0x7fff6b71b670 --> 0x10bb81050"Hello"
echo cast[ptr string](t3)[]
# --> Hello
# The following line doesn't compile:
echo repr(addr(t1))
# Error: expression has no address
```

### The unsafeAddr operator

For easier interoperability with other compiled languages such as C,
retrieving the address of a `let` variable, a parameter, or a `for` loop
variable can be accomplished by using the `unsafeAddr` operation:

``` nim
let myArray = [1, 2, 3]
foreignProcThatTakesAnAddr(unsafeAddr myArray)
```
