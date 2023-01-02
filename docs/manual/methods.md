## Methods

Procedures always use static dispatch. Methods use dynamic dispatch. For
dynamic dispatch to work on an object it should be a reference type.

``` nim
type
Expression = ref object of RootObj ## abstract base class for an expression
Literal = ref object of Expression
x: int
PlusExpr = ref object of Expression
a, b: Expression
method eval(e: Expression): int {.base.} =
  # override this base method
  raise newException(CatchableError, "Method without implementation override")

method eval(e: Literal): int = return e.x

method eval(e: PlusExpr): int =
  # watch out: relies on dynamic binding
  result = eval(e.a) + eval(e.b)

proc newLit(x: int): Literal =
  new(result)
  result.x = x

proc newPlus(a, b: Expression): PlusExpr =
  new(result)
  result.a = a
  result.b = b

echo eval(newPlus(newPlus(newLit(1), newLit(2)), newLit(4)))
```

In the example the constructors `newLit` and `newPlus` are procs because
they should use static binding, but `eval` is a method because it
requires dynamic binding.

As can be seen in the example, base methods have to be annotated with
the `base`{.interpreted-text role="idx"} pragma. The `base` pragma also
acts as a reminder for the programmer that a base method `m` is used as
the foundation to determine all the effects that a call to `m` might
cause.

**Note**: Compile-time execution is not (yet) supported for methods.

**Note**: Starting from Nim 0.20, generic methods are deprecated.

### Multi-methods

**Note:** Starting from Nim 0.20, to use multi-methods one must
explicitly pass `--multimethods:on`{.interpreted-text role="option"}
when compiling.

In a multi-method, all parameters that have an object type are used for
the dispatching:

``` {.nim test="\"nim c --multiMethods:on $1\""}
type
  Thing = ref object of RootObj
  Unit = ref object of Thing
    x: int

method collide(a, b: Thing) {.inline.} =
  quit "to override!"

method collide(a: Thing, b: Unit) {.inline.} =
  echo "1"

method collide(a: Unit, b: Thing) {.inline.} =
  echo "2"

var a, b: Unit
new a
new b
collide(a, b) # output: 2
```

### Inhibit dynamic method resolution via procCall

Dynamic method resolution can be inhibited via the builtin
`system.procCall`{.interpreted-text role="idx"}. This is somewhat
comparable to the `super`{.interpreted-text role="idx"} keyword that
traditional OOP languages offer.

``` {.nim test="\"nim c $1\""}
type
  Thing = ref object of RootObj
  Unit = ref object of Thing
    x: int

method m(a: Thing) {.base.} =
  echo "base"

method m(a: Unit) =
  # Call the base method:
  procCall m(Thing(a))
  echo "1"
```