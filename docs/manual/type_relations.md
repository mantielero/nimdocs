## Type relations

The following section defines several relations on types that are needed
to describe the type checking done by the compiler.

### Type equality

Nim uses structural type equivalence for most types. Only for objects,
enumerations and distinct types and for generic types name equivalence
is used.

### Subtype relation

If object `a` inherits from `b`, `a` is a subtype of `b`.

This subtype relation is extended to the types `var`, `ref`, `ptr`. If
`A` is a subtype of `B` and `A` and `B` are `object` types then:

-   `var A` is a subtype of `var B`
-   `ref A` is a subtype of `ref B`
-   `ptr A` is a subtype of `ptr B`.

**Note**: In later versions of the language the subtype relation might
be changed to *require* the pointer indirection in order to prevent
\"object slicing\".

### Convertible relation

A type `a` is **implicitly** convertible to type `b` iff the following
algorithm returns true:

``` nim
proc isImplicitlyConvertible(a, b: PType): bool =
  if isSubtype(a, b):
    return true
  if isIntLiteral(a):
    return b in {int8, int16, int32, int64, int, uint, uint8, uint16,
                 uint32, uint64, float32, float64}
  case a.kind
  of int:     result = b in {int32, int64}
  of int8:    result = b in {int16, int32, int64, int}
  of int16:   result = b in {int32, int64, int}
  of int32:   result = b in {int64, int}
  of uint:    result = b in {uint32, uint64}
  of uint8:   result = b in {uint16, uint32, uint64}
  of uint16:  result = b in {uint32, uint64}
  of uint32:  result = b in {uint64}
  of float32: result = b in {float64}
  of float64: result = b in {float32}
  of seq:
    result = b == openArray and typeEquals(a.baseType, b.baseType)
  of array:
    result = b == openArray and typeEquals(a.baseType, b.baseType)
    if a.baseType == char and a.indexType.rangeA == 0:
      result = b == cstring
  of cstring, ptr:
    result = b == pointer
  of string:
    result = b == cstring
  of proc:
    result = typeEquals(a, b) or compatibleParametersAndEffects(a, b)
```

We used the predicate `typeEquals(a, b)` for the \"type equality\"
property and the predicate `isSubtype(a, b)` for the \"subtype
relation\". `compatibleParametersAndEffects(a, b)` is currently not
specified.

Implicit conversions are also performed for Nim\'s `range` type
constructor.

Let `a0`, `b0` of type `T`.

Let `A = range[a0..b0]` be the argument\'s type, `F` the formal
parameter\'s type. Then an implicit conversion from `A` to `F` exists if
`a0 >= low(F) and b0 <= high(F)` and both `T` and `F` are signed
integers or if both are unsigned integers.

A type `a` is **explicitly** convertible to type `b` iff the following
algorithm returns true:

``` nim
proc isIntegralType(t: PType): bool =
result = isOrdinal(t) or t.kind in {float, float32, float64}
proc isExplicitlyConvertible(a, b: PType): bool =
  result = false
  if isImplicitlyConvertible(a, b): return true
  if typeEquals(a, b): return true
  if a == distinct and typeEquals(a.baseType, b): return true
  if b == distinct and typeEquals(b.baseType, a): return true
  if isIntegralType(a) and isIntegralType(b): return true
  if isSubtype(a, b) or isSubtype(b, a): return true
```

The convertible relation can be relaxed by a user-defined type
`converter`{.interpreted-text role="idx"}.

``` nim
converter toInt(x: char): int = result = ord(x)
var
  x: int
  chr: char = 'a'

# implicit conversion magic happens here
x = chr
echo x # => 97
# one can use the explicit form too
x = chr.toInt
echo x # => 97
```

The type conversion `T(a)` is an L-value if `a` is an L-value and
`typeEqualsOrDistinct(T, typeof(a))` holds.

### Assignment compatibility

An expression `b` can be assigned to an expression `a` iff `a` is an
`l-value` and `isImplicitlyConvertible(b.typ, a.typ)` holds.


