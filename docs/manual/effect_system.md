## Effect system

**Note**: The rules for effect tracking changed with the release of
version 1.6 of the Nim compiler. This section describes the new rules
that are activated via `--experimental:strictEffects`.

### Exception tracking

Nim supports exception tracking. The `raises`{.interpreted-text
role="idx"} pragma can be used to explicitly define which exceptions a
proc/iterator/method/converter is allowed to raise. The compiler
verifies this:

``` {.nim test="\"nim c $1\""}
proc p(what: bool) {.raises: [IOError, OSError].} =
  if what: raise newException(IOError, "IO")
  else: raise newException(OSError, "OS")
```

An empty `raises` list (`raises: []`) means that no exception may be
raised:

``` nim
proc p(): bool {.raises: [].} =
try:
unsafeCall()
result = true
except CatchableError:
result = false
```

A `raises` list can also be attached to a proc type. This affects type
compatibility:

``` {.nim test="\"nim c $1\"" status="1"}
type
  Callback = proc (s: string) {.raises: [IOError].}
var
  c: Callback

proc p(x: string) =
  raise newException(OSError, "OS")

c = p # type error
```

For a routine `p`, the compiler uses inference rules to determine the
set of possibly raised exceptions; the algorithm operates on `p`\'s call
graph:

1.  Every indirect call via some proc type `T` is assumed to raise
    `system.Exception` (the base type of the exception hierarchy) and
    thus any exception unless `T` has an explicit `raises` list.
    However, if the call is of the form `f(...)` where `f` is a
    parameter of the currently analyzed routine it is ignored that is
    marked as `.effectsOf: f`. The call is optimistically assumed to
    have no effect. Rule 2 compensates for this case.
2.  Every expression `e` of some proc type within a call that is passed
    to parameter marked as `.effectsOf` is assumed to be called
    indirectly and thus its raises list is added to `p`\'s raises list.
3.  Every call to a proc `q` which has an unknown body (due to a forward
    declaration) is assumed to raise `system.Exception` unless `q` has
    an explicit `raises` list. Procs that are `importc`\'ed are assumed
    to have `.raises: []`, unless explicitly declared otherwise.
4.  Every call to a method `m` is assumed to raise `system.Exception`
    unless `m` has an explicit `raises` list.
5.  For every other call, the analysis can determine an exact `raises`
    list.
6.  For determining a `raises` list, the `raise` and `try` statements of
    `p` are taken into consideration.

Exceptions inheriting from `system.Defect` are not tracked with the
`.raises: []` exception tracking mechanism. This is more consistent with
the built-in operations. The following code is valid:

``` nim
proc mydiv(a, b): int {.raises: [].} =
  a div b # can raise an DivByZeroDefect
```

And so is:

``` nim
proc mydiv(a, b): int {.raises: [].} =
  if b == 0: raise newException(DivByZeroDefect, "division by zero")
  else: result = a div b
```

The reason for this is that `DivByZeroDefect` inherits from `Defect` and
with `--panics:on`{.interpreted-text role="option"} Defects become
unrecoverable errors. (Since version 1.4 of the language.)

### EffectsOf annotation

Rules 1-2 of the exception tracking inference rules (see the previous
section) ensure the following works:

``` nim
proc weDontRaiseButMaybeTheCallback(callback: proc()) {.raises: [], effectsOf: callback.} =
callback()
proc doRaise() {.raises: [IOError].} =
  raise newException(IOError, "IO")

proc use() {.raises: [].} =
  # doesn't compile! Can raise IOError!
  weDontRaiseButMaybeTheCallback(doRaise)
```

As can be seen from the example, a parameter of type `proc (...)` can be
annotated as `.effectsOf`. Such a parameter allows for effect
polymorphism: The proc `weDontRaiseButMaybeTheCallback` raises the
exceptions that `callback` raises.

So in many cases a callback does not cause the compiler to be overly
conservative in its effect analysis:

``` {.nim test="\"nim c $1\"" status="1"}
{.push warningAsError[Effect]: on.}
{.experimental: "strictEffects".}

import algorithm

type
  MyInt = distinct int

var toSort = @[MyInt 1, MyInt 2, MyInt 3]

proc cmpN(a, b: MyInt): int =
  cmp(a.int, b.int)

proc harmless {.raises: [].} =
  toSort.sort cmpN

proc cmpE(a, b: MyInt): int {.raises: [Exception].} =
  cmp(a.int, b.int)

proc harmfull {.raises: [].} =
  # does not compile, `sort` can now raise Exception
  toSort.sort cmpE
```

### Tag tracking

Exception tracking is part of Nim\'s `effect system`{.interpreted-text
role="idx"}. Raising an exception is an *effect*. Other effects can also
be defined. A user defined effect is a means to *tag* a routine and to
perform checks against this tag:

``` {.nim test="\"nim c --warningAsError:Effect:on $1\"" status="1"}
type IO = object ## input/output effect
proc readLine(): string {.tags: [IO].} = discard

proc no_IO_please() {.tags: [].} =
  # the compiler prevents this:
  let x = readLine()
```

A tag has to be a type name. A `tags` list - like a `raises` list - can
also be attached to a proc type. This affects type compatibility.

The inference for tag tracking is analogous to the inference for
exception tracking.

### Side effects

The `noSideEffect` pragma is used to mark a proc/iterator that can have
only side effects through parameters. This means that the proc/iterator
only changes locations that are reachable from its parameters and the
return value only depends on the parameters. If none of its parameters
have the type `var`, `ref`, `ptr`, `cstring`, or `proc`, then no
locations are modified.

In other words, a routine has no side effects if it does not access a
threadlocal or global variable and it does not call any routine that has
a side effect.

It is a static error to mark a proc/iterator to have no side effect if
the compiler cannot verify this.

As a special semantic rule, the built-in
[debugEcho](system.html#debugEcho,varargs%5Btyped,%5D) pretends to be
free of side effects so that it can be used for debugging routines
marked as `noSideEffect`.

`func` is syntactic sugar for a proc with no side effects:

``` nim
func `+` (x, y: int): int
```

To override the compiler\'s side effect analysis a `{.noSideEffect.}`
`cast` pragma block can be used:

``` nim
func f() =
  {.cast(noSideEffect).}:
    echo "test"
```

**Side effects are usually inferred. The inference for side effects is
analogous to the inference for exception tracking.**

### GC safety effect

We call a proc `p` `GC safe` when it
doesn\'t access any global variable that contains GC\'ed memory
(`string`, `seq`, `ref` or a closure) either directly or indirectly
through a call to a GC unsafe proc.

**The GC safety property is usually inferred. The inference for GC
safety is analogous to the inference for exception tracking.**

The `gcsafe` annotation can be used to
mark a proc to be gcsafe, otherwise this property is inferred by the
compiler. Note that `noSideEffect` implies `gcsafe`.

Routines that are imported from C are always assumed to be `gcsafe`.

To override the compiler\'s gcsafety analysis a `{.cast(gcsafe).}`
pragma block can be used:

``` nim
var
  someGlobal: string = "some string here"
  perThread {.threadvar.}: string

proc setPerThread() =
  {.cast(gcsafe).}:
    deepCopy(perThread, someGlobal)
```

See also:

-   [Shared heap memory management](mm.html).

### Effects pragma

The `effects` pragma has been designed to assist the programmer with the
effects analysis. It is a statement that makes the compiler output all
inferred effects up to the `effects`\'s position:

``` nim
proc p(what: bool) =
if what:
raise newException(IOError, "IO")
{.effects.}
else:
raise newException(OSError, "OS")
```

The compiler produces a hint message that `IOError` can be raised.
`OSError` is not listed as it cannot be raised in the branch the
`effects` pragma appears in.