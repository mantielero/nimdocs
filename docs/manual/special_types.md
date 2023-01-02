## Special Types

### static\[T\]

As their name suggests, static parameters must be constant expressions:

``` nim
proc precompiledRegex(pattern: static string): RegEx =
  var res {.global.} = re(pattern)
  return res

precompiledRegex("/d+") # Replaces the call with a precompiled
                        # regex, stored in a global variable

precompiledRegex(paramStr(1)) # Error, command-line options
                              # are not constant expressions
```

For the purposes of code generation, all static params are treated as
generic params - the proc will be compiled separately for each unique
supplied value (or combination of values).

Static params can also appear in the signatures of generic types:

``` nim
type
  Matrix[M,N: static int; T: Number] = array[0..(M*N - 1), T]
    # Note how `Number` is just a type constraint here, while
    # `static int` requires us to supply an int value

  AffineTransform2D[T] = Matrix[3, 3, T]
  AffineTransform3D[T] = Matrix[4, 4, T]

var m1: AffineTransform3D[float]  # OK
var m2: AffineTransform2D[string] # Error, `string` is not a `Number`
```

Please note that `static T` is just a syntactic convenience for the
underlying generic type `static[T]`. The type param can be omitted to
obtain the type class of all constant expressions. A more specific type
class can be created by instantiating `static` with another type class.

One can force an expression to be evaluated at compile time as a
constant expression by coercing it to a corresponding `static` type:

``` nim
import std/math
echo static(fac(5)), " ", static[bool](16.isPowerOfTwo)
```

The compiler will report any failure to evaluate the expression or a
possible type mismatch error.

### typedesc\[T\]

In many contexts, Nim treats the names of types as regular values. These
values exist only during the compilation phase, but since all values
must have a type, `typedesc` is considered their special type.

`typedesc` acts as a generic type. For instance, the type of the symbol
`int` is `typedesc[int]`. Just like with regular generic types, when the
generic param is omitted, `typedesc` denotes the type class of all
types. As a syntactic convenience, one can also use `typedesc` as a
modifier.

Procs featuring `typedesc` params are considered implicitly generic.
They will be instantiated for each unique combination of supplied types,
and within the body of the proc, the name of each param will refer to
the bound concrete type:

``` nim
proc new(T: typedesc): ref T =
  echo "allocating ", T.name
  new(result)

var n = Node.new
var tree = new(BinaryTree[int])
```

When multiple type params are present, they will bind freely to
different types. To force a bind-once behavior, one can use an explicit
generic param:

``` nim
proc acceptOnlyTypePairs[T, U](A, B: typedesc[T]; C, D: typedesc[U])
```

Once bound, type params can appear in the rest of the proc signature:

``` {.nim test="\"nim c $1\""}
template declareVariableWithType(T: typedesc, value: T) =
  var x: T = value

declareVariableWithType int, 42
```

Overload resolution can be further influenced by constraining the set of
types that will match the type param. This works in practice by
attaching attributes to types via templates. The constraint can be a
concrete type or a type class.

``` {.nim test="\"nim c $1\""}
template maxval(T: typedesc[int]): int = high(int)
template maxval(T: typedesc[float]): float = Inf

var i = int.maxval
var f = float.maxval
when false:
  var s = string.maxval # error, maxval is not implemented for string

template isNumber(t: typedesc[object]): string = "Don't think so."
template isNumber(t: typedesc[SomeInteger]): string = "Yes!"
template isNumber(t: typedesc[SomeFloat]): string = "Maybe, could be NaN."

echo "is int a number? ", isNumber(int)
echo "is float a number? ", isNumber(float)
echo "is RootObj a number? ", isNumber(RootObj)
```

Passing `typedesc` is almost identical, just with the difference that
the macro is not instantiated generically. The type expression is simply
passed as a `NimNode` to the macro, like everything else.

``` nim
import std/macros

macro forwardType(arg: typedesc): typedesc =
  # `arg` is of type `NimNode`
  let tmp: NimNode = arg
  result = tmp

var tmp: forwardType(int)
```

### typeof operator

**Note**: `typeof(x)` can for historical reasons also be written as
`type(x)` but `type(x)` is discouraged.

One can obtain the type of a given expression by constructing a `typeof`
value from it (in many other languages this is known as the
`typeof`{.interpreted-text role="idx"} operator):

``` nim
var x = 0
var y: typeof(x) # y has type int
```

If `typeof` is used to determine the result type of a
proc/iterator/converter call `c(X)` (where `X` stands for a possibly
empty list of arguments), the interpretation, where `c` is an iterator,
is preferred over the other interpretations, but this behavior can be
changed by passing `typeOfProc` as the second argument to \`typeof\`:

``` {.nim test="\"nim c $1\""}
iterator split(s: string): string = discard
proc split(s: string): seq[string] = discard

# since an iterator is the preferred interpretation, `y` has the type `string`:
assert typeof("a b c".split) is string

assert typeof("a b c".split, typeOfProc) is seq[string]
```
