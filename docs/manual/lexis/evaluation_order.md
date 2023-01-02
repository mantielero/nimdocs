## Order of evaluation

Order of evaluation is strictly left-to-right, inside-out as it is
typical for most others imperative programming languages:

```nim
var s = ""

proc p(arg: int): int =
  s.add $arg
  result = arg

discard p(p(1) + p(2))

doAssert s == "123"
```

Assignments are not special, the left-hand-side expression is evaluated
before the right-hand side:

```nim
var v = 0
proc getI(): int =
  result = v
  inc v

var a, b: array[0..2, int]

proc someCopy(a: var int; b: int) = a = b

a[getI()] = getI()

doAssert a == [1, 0, 0]

v = 0
someCopy(b[getI()], getI())

doAssert b == [1, 0, 0]
```

Rationale: Consistency with overloaded assignment or assignment-like
operations, `a = b` can be read as `performSomeCopy(a, b)`.

However, the concept of \"order of evaluation\" is only applicable after
the code was normalized: The normalization involves template expansions
and argument reorderings that have been passed to named parameters:

```nim
var s = ""

proc p(): int =
  s.add "p"
  result = 5

proc q(): int =
  s.add "q"
  result = 3

# Evaluation order is 'b' before 'a' due to template
# expansion's semantics.
template swapArgs(a, b): untyped =
  b + a

doAssert swapArgs(p() + q(), q() - p()) == 6
doAssert s == "qppq"

# Evaluation order is not influenced by named parameters:
proc construct(first, second: int) =
  discard

# 'p' is evaluated before 'q'!
construct(second = q(), first = p())

doAssert s == "qppqpq"
```

Rationale: This is far easier to implement than hypothetical
alternatives.

