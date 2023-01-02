## Templates

A template is a simple form of a macro: It is a simple substitution
mechanism that operates on Nim\'s abstract syntax trees. It is processed
in the semantic pass of the compiler.

The syntax to *invoke* a template is the same as calling a procedure.

Example:

``` nim
template `!=` (a, b: untyped): untyped =
# this definition exists in the System module
not (a == b)
assert(5 != 6) # the compiler rewrites that to: assert(not (5 == 6))
```

The `!=`, `>`, `>=`, `in`, `notin`, `isnot` operators are in fact
templates:

| `a > b` is transformed into `b < a`.
| `a in b` is transformed into `contains(b, a)`.
| `notin` and `isnot` have the obvious meanings.

The \"types\" of templates can be the symbols `untyped`, `typed` or
`typedesc`. These are \"meta types\", they can only be used in certain
contexts. Regular types can be used too; this implies that `typed`
expressions are expected.

### Typed vs untyped parameters

An `untyped` parameter means that symbol lookups and type resolution is
not performed before the expression is passed to the template. This
means that *undeclared* identifiers, for example, can be passed to the
template:

``` {.nim test="\"nim c $1\""}
template declareInt(x: untyped) =
  var x: int

declareInt(x) # valid
x = 3
```

``` {.nim test="\"nim c $1\"" status="1"}
template declareInt(x: typed) =
  var x: int

declareInt(x) # invalid, because x has not been declared and so it has no type
```

A template where every parameter is `untyped` is called an
`immediate` template. For historical
reasons, templates can be explicitly annotated with an `immediate`
pragma and then these templates do not take part in overloading
resolution and the parameters\' types are *ignored* by the compiler.
Explicit immediate templates are now deprecated.

**Note**: For historical reasons, `stmt` was an alias for `typed` and
`expr` was an alias for `untyped`, but they are removed.

### Passing a code block to a template

One can pass a block of statements as the last argument to a template
following the special `:` syntax:

``` {.nim test="\"nim c $1\""}
template withFile(f, fn, mode, actions: untyped): untyped =
  var f: File
  if open(f, fn, mode):
    try:
      actions
    finally:
      close(f)
  else:
    quit("cannot open: " & fn)

withFile(txt, "ttempl3.txt", fmWrite):  # special colon
  txt.writeLine("line 1")
  txt.writeLine("line 2")
```

In the example, the two `writeLine` statements are bound to the
`actions` parameter.

Usually, to pass a block of code to a template, the parameter that
accepts the block needs to be of type `untyped`. Because symbol lookups
are then delayed until template instantiation time:

``` {.nim test="\"nim c $1\"" status="1"}
template t(body: typed) =
  proc p = echo "hey"
  block:
    body

t:
  p()  # fails with 'undeclared identifier: p'
```

The above code fails with the error message that `p` is not declared.
The reason for this is that the `p()` body is type-checked before
getting passed to the `body` parameter and type checking in Nim implies
symbol lookups. The same code works with `untyped` as the passed body is
not required to be type-checked:

``` {.nim test="\"nim c $1\""}
template t(body: untyped) =
  proc p = echo "hey"
  block:
    body

t:
  p()  # compiles
```

### Varargs of untyped

In addition to the `untyped` meta-type that prevents type checking,
there is also `varargs[untyped]` so that not even the number of
parameters is fixed:

``` {.nim test="\"nim c $1\""}
template hideIdentifiers(x: varargs[untyped]) = discard

hideIdentifiers(undeclared1, undeclared2)
```

However, since a template cannot iterate over varargs, this feature is
generally much more useful for macros.

### Symbol binding in templates

A template is a `hygienic` macro and so
opens a new scope. Most symbols are bound from the definition scope of
the template:

``` nim
# Module A
var
lastId = 0
template genId*: untyped =
  inc(lastId)
  lastId
```

``` nim
# Module B
import A
echo genId() # Works as 'lastId' has been bound in 'genId's defining scope
```

As in generics, symbol binding can be influenced via `mixin` or `bind`
statements.

### Identifier construction

In templates, identifiers can be constructed with the backticks
notation:

``` {.nim test="\"nim c $1\""}
template typedef(name: untyped, typ: typedesc) =
  type
    `T name`* {.inject.} = typ
    `P name`* {.inject.} = ref `T name`

typedef(myint, int)
var x: PMyInt
```

In the example, `name` is instantiated with `myint`, so \`T name\`
becomes `Tmyint`.

### Lookup rules for template parameters

A parameter `p` in a template is even substituted in the expression
`x.p`. Thus, template arguments can be used as field names and a global
symbol can be shadowed by the same argument name even when fully
qualified:

``` nim
# module 'm'
type
  Lev = enum
    levA, levB

var abclev = levB

template tstLev(abclev: Lev) =
  echo abclev, " ", m.abclev

tstLev(levA)
# produces: 'levA levA'
```

But the global symbol can properly be captured by a `bind` statement:

``` nim
# module 'm'
type
  Lev = enum
    levA, levB

var abclev = levB

template tstLev(abclev: Lev) =
  bind m.abclev
  echo abclev, " ", m.abclev

tstLev(levA)
# produces: 'levA levB'
```

### Hygiene in templates

Per default, templates are `hygienic`:
Local identifiers declared in a template cannot be accessed in the
instantiation context:

``` {.nim test="\"nim c $1\""}
template newException*(exceptn: typedesc, message: string): untyped =
  var
    e: ref exceptn  # e is implicitly gensym'ed here
  new(e)
  e.msg = message
  e

# so this works:
let e = "message"
raise newException(IoError, e)
```

Whether a symbol that is declared in a template is exposed to the
instantiation scope is controlled by the `inject`{.interpreted-text
role="idx"} and `gensym` pragmas:
`gensym`\'ed symbols are not exposed but `inject`\'ed symbols are.

The default for symbols of entity `type`, `var`, `let` and `const` is
`gensym` and for `proc`, `iterator`, `converter`, `template`, `macro` is
`inject`. However, if the name of the entity is passed as a template
parameter, it is an `inject`\'ed symbol:

``` nim
template withFile(f, fn, mode: untyped, actions: untyped): untyped =
block:
var f: File  # since 'f' is a template param, it's injected implicitly
...
withFile(txt, "ttempl3.txt", fmWrite):
  txt.writeLine("line 1")
  txt.writeLine("line 2")
```

The `inject` and `gensym` pragmas are second class annotations; they
have no semantics outside of a template definition and cannot be
abstracted over:

``` nim
{.pragma myInject: inject.}
template t() =
  var x {.myInject.}: int # does NOT work
```

To get rid of hygiene in templates, one can use the
`dirty` pragma for a template. `inject`
and `gensym` have no effect in `dirty` templates.

`gensym`\'ed symbols cannot be used as `field` in the `x.field` syntax.
Nor can they be used in the `ObjectConstruction(field: value)` and
`namedParameterCall(field = value)` syntactic constructs.

The reason for this is that code like

``` {.nim test="\"nim c $1\""}
type
  T = object
    f: int

template tmp(x: T) =
  let f = 34
  echo x.f, T(f: 4)
```

should work as expected.

However, this means that the method call syntax is not available for
`gensym`\'ed symbols:

``` {.nim test="\"nim c $1\"" status="1"}
template tmp(x) =
  type
    T {.gensym.} = int

  echo x.T # invalid: instead use:  'echo T(x)'.

tmp(12)
```

**Note**: The Nim compiler prior to version 1 was more lenient about
this requirement. Use the `--useVersion:0.19`{.interpreted-text
role="option"} switch for a transition period.

### Limitations of the method call syntax

The expression `x` in `x.f` needs to be semantically checked (that means
symbol lookup and type checking) before it can be decided that it needs
to be rewritten to `f(x)`. Therefore the dot syntax has some limitations
when it is used to invoke templates/macros:

``` {.nim test="\"nim c $1\"" status="1"}
template declareVar(name: untyped) =
  const name {.inject.} = 45

# Doesn't compile:
unknownIdentifier.declareVar
```

It is also not possible to use fully qualified identifiers with module
symbol in method call syntax. The order in which the dot operator binds
to symbols prohibits this.

``` {.nim test="\"nim c $1\"" status="1"}
import std/sequtils

var myItems = @[1,3,3,7]
let N1 = count(myItems, 3) # OK
let N2 = sequtils.count(myItems, 3) # fully qualified, OK
let N3 = myItems.count(3) # OK
let N4 = myItems.sequtils.count(3) # illegal, `myItems.sequtils` can't be resolved
```

This means that when for some reason a procedure needs a disambiguation
through the module name, the call needs to be written in function call
syntax.
