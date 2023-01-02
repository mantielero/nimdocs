## Types

All expressions have a type that is known during semantic analysis. Nim
is statically typed. One can declare new types, which is in essence
defining an identifier that can be used to denote this custom type.

These are the major type classes:

-   ordinal types (consist of integer, bool, character, enumeration (and
    subranges thereof) types)
-   floating-point types
-   string type
-   structured types
-   reference (pointer) type
-   procedural type
-   generic type

### Ordinal types

Ordinal types have the following characteristics:

-   Ordinal types are countable and ordered. This property allows the
    operation of functions such as `inc`, `ord`, and `dec` on ordinal
    types to be defined.
-   Ordinal types have a smallest possible value, accessible with
    `low(type)`. Trying to count further down than the smallest value
    produces a panic or a static error.
-   Ordinal types have a largest possible value, accessible with
    `high(type)`. Trying to count further up than the largest value
    produces a panic or a static error.

Integers, bool, characters, and enumeration types (and subranges of
these types) belong to ordinal types.

A distinct type is an ordinal type if its base type is an ordinal type.

### Pre-defined integer types

These integer types are pre-defined:

`int`

:   the generic signed integer type; its size is platform-dependent and
    has the same size as a pointer. This type should be used in general.
    An integer literal that has no type suffix is of this type if it is
    in the range `low(int32)..high(int32)` otherwise the literal\'s type
    is `int64`.

`int`XX

:   additional signed integer types of XX bits use this naming scheme
    (example: int16 is a 16-bit wide integer). The current
    implementation supports `int8`, `int16`, `int32`, `int64`. Literals
    of these types have the suffix \'iXX.

`uint`

:   the generic `unsigned integer`{.interpreted-text role="idx"} type;
    its size is platform-dependent and has the same size as a pointer.
    An integer literal with the type suffix `'u` is of this type.

`uint`XX

:   additional unsigned integer types of XX bits use this naming scheme
    (example: uint16 is a 16-bit wide unsigned integer). The current
    implementation supports `uint8`, `uint16`, `uint32`, `uint64`.
    Literals of these types have the suffix \'uXX. Unsigned operations
    all wrap around; they cannot lead to over- or underflow errors.

In addition to the usual arithmetic operators for signed and unsigned
integers (`+ - *` etc.) there are also operators that formally work on
*signed* integers but treat their arguments as *unsigned*: They are
mostly provided for backwards compatibility with older versions of the
language that lacked unsigned integer types. These unsigned operations
for signed integers use the `%` suffix as convention:

  operation    meaning
  ------------ -------------------------------------------------------------------------------------------------------
  `a +% b`     unsigned integer addition
  `a -% b`     unsigned integer subtraction
  `a *% b`     unsigned integer multiplication
  `a /% b`     unsigned integer division
  `a %% b`     unsigned integer modulo operation
  `a <% b`     treat `a` and `b` as unsigned and compare
  `a <=% b`    treat `a` and `b` as unsigned and compare
  `ze(a)`      extends the bits of `a` with zeros until it has the width of the `int` type
  `toU8(a)`    treats `a` as unsigned and converts it to an unsigned integer of 8 bits (but still the `int8` type)
  `toU16(a)`   treats `a` as unsigned and converts it to an unsigned integer of 16 bits (but still the `int16` type)
  `toU32(a)`   treats `a` as unsigned and converts it to an unsigned integer of 32 bits (but still the `int32` type)

`Automatic type conversion`{.interpreted-text role="idx"} is performed
in expressions where different kinds of integer types are used: the
smaller type is converted to the larger.

A `narrowing type conversion`{.interpreted-text role="idx"} converts a
larger to a smaller type (for example `int32 -> int16`). A
`widening type conversion`{.interpreted-text role="idx"} converts a
smaller type to a larger type (for example `int16 -> int32`). In Nim
only widening type conversions are *implicit*:

``` nim
var myInt16 = 5i16
var myInt: int
myInt16 + 34     # of type `int16`
myInt16 + myInt  # of type `int`
myInt16 + 2i32   # of type `int32`
```

However, `int` literals are implicitly convertible to a smaller integer
type if the literal\'s value fits this smaller type and such a
conversion is less expensive than other implicit conversions, so
`myInt16 + 34` produces an `int16` result.

For further details, see [Convertible
relation](#type-relations-convertible-relation).

### Subrange types

A subrange type is a range of values from an ordinal or floating-point
type (the base type). To define a subrange type, one must specify its
limiting values \-- the lowest and highest value of the type. For
example:

``` nim
type
Subrange = range[0..5]
PositiveFloat = range[0.0..Inf]
Positive* = range[1..high(int)] # as defined in `system`
```

`Subrange` is a subrange of an integer which can only hold the values 0
to 5. `PositiveFloat` defines a subrange of all positive floating-point
values. NaN does not belong to any subrange of floating-point types.
Assigning any other value to a variable of type `Subrange` is a panic
(or a static error if it can be determined during semantic analysis).
Assignments from the base type to one of its subrange types (and vice
versa) are allowed.

A subrange type has the same size as its base type (`int` in the
Subrange example).

### Pre-defined floating-point types

The following floating-point types are pre-defined:

`float`

:   the generic floating-point type; its size used to be
    platform-dependent, but now it is always mapped to `float64`. This
    type should be used in general.

`float`XX

:   an implementation may define additional floating-point types of XX
    bits using this naming scheme (example: `float64` is a 64-bit wide
    float). The current implementation supports `float32` and `float64`.
    Literals of these types have the suffix \'fXX.

Automatic type conversion in expressions with different kinds of
floating-point types is performed: See [Convertible
relation](#convertible-relation) for further details. Arithmetic
performed on floating-point types follows the IEEE standard. Integer
types are not converted to floating-point types automatically and vice
versa.

The IEEE standard defines five types of floating-point exceptions:

-   Invalid: operations with mathematically invalid operands, for
    example 0.0/0.0, sqrt(-1.0), and log(-37.8).
-   Division by zero: divisor is zero and dividend is a finite nonzero
    number, for example 1.0/0.0.
-   Overflow: operation produces a result that exceeds the range of the
    exponent, for example MAXDOUBLE+0.0000000000001e308.
-   Underflow: operation produces a result that is too small to be
    represented as a normal number, for example, MINDOUBLE \* MINDOUBLE.
-   Inexact: operation produces a result that cannot be represented with
    infinite precision, for example, 2.0 / 3.0, log(1.1) and 0.1 in
    input.

The IEEE exceptions are either ignored during execution or mapped to the
Nim exceptions: `FloatInvalidOpDefect`{.interpreted-text role="idx"},
`FloatDivByZeroDefect`{.interpreted-text role="idx"},
`FloatOverflowDefect`{.interpreted-text role="idx"},
`FloatUnderflowDefect`{.interpreted-text role="idx"}, and
`FloatInexactDefect`{.interpreted-text role="idx"}. These exceptions
inherit from the `FloatingPointDefect`{.interpreted-text role="idx"}
base class.

Nim provides the pragmas `nanChecks`{.interpreted-text role="idx"} and
`infChecks`{.interpreted-text role="idx"} to control whether the IEEE
exceptions are ignored or trap a Nim exception:

``` nim
{.nanChecks: on, infChecks: on.}
var a = 1.0
var b = 0.0
echo b / b # raises FloatInvalidOpDefect
echo a / b # raises FloatOverflowDefect
```

In the current implementation `FloatDivByZeroDefect` and
`FloatInexactDefect` are never raised. `FloatOverflowDefect` is raised
instead of `FloatDivByZeroDefect`. There is also a
`floatChecks`{.interpreted-text role="idx"} pragma that is a short-cut
for the combination of `nanChecks` and `infChecks` pragmas.
`floatChecks` are turned off as default.

The only operations that are affected by the `floatChecks` pragma are
the `+`, `-`, `*`, `/` operators for floating-point types.

An implementation should always use the maximum precision available to
evaluate floating-point values during semantic analysis; this means
expressions like `0.09'f32 + 0.01'f32 == 0.09'f64 + 0.01'f64` that are
evaluating during constant folding are true.

### Boolean type

The boolean type is named `bool`{.interpreted-text role="idx"} in Nim
and can be one of the two pre-defined values `true` and `false`.
Conditions in `while`, `if`, `elif`, `when`-statements need to be of
type `bool`.

This condition holds:

    ord(false) == 0 and ord(true) == 1

The operators `not, and, or, xor, <, <=, >, >=, !=, ==` are defined for
the bool type. The `and` and `or` operators perform short-cut
evaluation. Example:

``` nim
while p != nil and p.name != "xyz":
  # p.name is not evaluated if p == nil
  p = p.next
```

The size of the bool type is one byte.

### Character type

The character type is named `char` in Nim. Its size is one byte. Thus it
cannot represent a UTF-8 character, but a part of it.

The `Rune` type is used for Unicode characters, it can represent any
Unicode character. `Rune` is declared in the [unicode
module](unicode.html).

### Enumeration types

Enumeration types define a new type whose values consist of the ones
specified. The values are ordered. Example:

``` nim
type
  Direction = enum
    north, east, south, west
```

Now the following holds:

    ord(north) == 0
    ord(east) == 1
    ord(south) == 2
    ord(west) == 3

    # Also allowed:
    ord(Direction.west) == 3

The implied order is: north \< east \< south \< west. The comparison
operators can be used with enumeration types. Instead of `north` etc,
the enum value can also be qualified with the enum type that it resides
in, `Direction.north`.

For better interfacing to other programming languages, the fields of
enum types can be assigned an explicit ordinal value. However, the
ordinal values have to be in ascending order. A field whose ordinal
value is not explicitly given is assigned the value of the previous
field + 1.

An explicit ordered enum can have *holes*:

``` nim
type
TokenType = enum
a = 2, b = 4, c = 89 # holes are valid
```

However, it is then not ordinal anymore, so it is impossible to use
these enums as an index type for arrays. The procedures `inc`, `dec`,
`succ` and `pred` are not available for them either.

The compiler supports the built-in stringify operator `$` for
enumerations. The stringify\'s result can be controlled by explicitly
giving the string values to use:

``` nim
type
  MyEnum = enum
    valueA = (0, "my value A"),
    valueB = "value B",
    valueC = 2,
    valueD = (3, "abc")
```

As can be seen from the example, it is possible to both specify a
field\'s ordinal value and its string value by using a tuple. It is also
possible to only specify one of them.

An enum can be marked with the `pure` pragma so that its fields are
added to a special module-specific hidden scope that is only queried as
the last attempt. Only non-ambiguous symbols are added to this scope.
But one can always access these via type qualification written as
\`MyEnum.value\`:

``` nim
type
  MyEnum {.pure.} = enum
    valueA, valueB, valueC, valueD, amb

  OtherEnum {.pure.} = enum
    valueX, valueY, valueZ, amb


echo valueA # MyEnum.valueA
echo amb    # Error: Unclear whether it's MyEnum.amb or OtherEnum.amb
echo MyEnum.amb # OK.
```

To implement bit fields with enums see [Bit
fields](#set-type-bit-fields)

### Overloadable enum field names

To be enabled via `{.experimental: "overloadableEnums".}`.

Enum field names are overloadable much like routines. When an overloaded
enum field is used, it produces a closed sym choice construct, here
written as `(E|E)`. During overload resolution the right `E` is picked,
if possible. For (array/object\...) constructors the right `E` is
picked, comparable to how `[byte(1), 2, 3]` works, one needs to use
`[T.E, E2, E3]`. Ambiguous enum fields produce a static error:

``` {.nim test="\"nim c $1\""}
{.experimental: "overloadableEnums".}

type
  E1 = enum
    value1,
    value2
  E2 = enum
    value1,
    value2 = 4

const
  Lookuptable = [
    E1.value1: "1",
    value2: "2"
  ]

proc p(e: E1) =
  # disambiguation in 'case' statements:
  case e
  of value1: echo "A"
  of value2: echo "B"

p value2
```

### String type

All string literals are of the type `string`. A string in Nim is very
similar to a sequence of characters. However, strings in Nim are both
zero-terminated and have a length field. One can retrieve the length
with the builtin `len` procedure; the length never counts the
terminating zero.

The terminating zero cannot be accessed unless the string is converted
to the `cstring` type first. The terminating zero assures that this
conversion can be done in O(1) and without any allocations.

The assignment operator for strings always copies the string. The `&`
operator concatenates strings.

Most native Nim types support conversion to strings with the special `$`
proc. When calling the `echo` proc, for example, the built-in stringify
operation for the parameter is called:

``` nim
echo 3 # calls `$` for `int`
```

Whenever a user creates a specialized object, implementation of this
procedure provides for `string` representation.

``` nim
type
Person = object
name: string
age: int
proc `$`(p: Person): string = # `$` always returns a string
  result = p.name & " is " &
          $p.age & # we *need* the `$` in front of p.age which
                   # is natively an integer to convert it to
                   # a string
          " years old."
```

While `$p.name` can also be used, the `$` operation on a string does
nothing. Note that we cannot rely on automatic conversion from an `int`
to a `string` like we can for the `echo` proc.

Strings are compared by their lexicographical order. All comparison
operators are available. Strings can be indexed like arrays (lower bound
is 0). Unlike arrays, they can be used in case statements:

``` nim
case paramStr(i)
of "-v": incl(options, optVerbose)
of "-h", "-?": incl(options, optHelp)
else: write(stdout, "invalid command line option!\n")
```

Per convention, all strings are UTF-8 strings, but this is not enforced.
For example, when reading strings from binary files, they are merely a
sequence of bytes. The index operation `s[i]` means the i-th *char* of
`s`, not the i-th *unichar*. The iterator `runes` from the [unicode
module](unicode.html) can be used for iteration over all Unicode
characters.

### cstring type

The `cstring` type meaning `compatible string` is the native
representation of a string for the compilation backend. For the C
backend the `cstring` type represents a pointer to a zero-terminated
char array compatible with the type `char*` in Ansi C. Its primary
purpose lies in easy interfacing with C. The index operation `s[i]`
means the i-th *char* of `s`; however no bounds checking for `cstring`
is performed making the index operation unsafe.

A Nim `string` is implicitly convertible to `cstring` for convenience.
If a Nim string is passed to a C-style variadic proc, it is implicitly
converted to `cstring` too:

``` nim
proc printf(formatstr: cstring) {.importc: "printf", varargs,
header: "<stdio.h>".}
printf("This works %s", "as expected")
```

Even though the conversion is implicit, it is not *safe*: The garbage
collector does not consider a `cstring` to be a root and may collect the
underlying memory. For this reason, the implicit conversion will be
removed in future releases of the Nim compiler. Certain idioms like
conversion of a `const` string to `cstring` are safe and will remain to
be allowed.

A `$` proc is defined for cstrings that returns a string. Thus to get a
nim string from a cstring:

``` nim
var str: string = "Hello!"
var cstr: cstring = str
var newstr: string = $cstr
```

`cstring` literals shouldn\'t be modified.

``` nim
var x = cstring"literals"
x[1] = 'A' # This is wrong!!!
```

If the `cstring` originates from a regular memory (not read-only
memory), it can be modified:

``` nim
var x = "123456"
var s: cstring = x
s[0] = 'u' # This is ok
```

### Structured types

A variable of a structured type can hold multiple values at the same
time. Structured types can be nested to unlimited levels. Arrays,
sequences, tuples, objects, and sets belong to the structured types.

### Array and sequence types

Arrays are a homogeneous type, meaning that each element in the array
has the same type. Arrays always have a fixed length specified as a
constant expression (except for open arrays). They can be indexed by any
ordinal type. A parameter `A` may be an *open array*, in which case it
is indexed by integers from 0 to `len(A)-1`. An array expression may be
constructed by the array constructor `[]`. The element type of this
array expression is inferred from the type of the first element. All
other elements need to be implicitly convertible to this type.

An array type can be defined using the `array[size, T]` syntax, or using
`array[lo..hi, T]` for arrays that start at an index other than zero.

Sequences are similar to arrays but of dynamic length which may change
during runtime (like strings). Sequences are implemented as growable
arrays, allocating pieces of memory as items are added. A sequence `S`
is always indexed by integers from 0 to `len(S)-1` and its bounds are
checked. Sequences can be constructed by the array constructor `[]` in
conjunction with the array to sequence operator `@`. Another way to
allocate space for a sequence is to call the built-in `newSeq`
procedure.

A sequence may be passed to a parameter that is of type *open array*.

Example:

``` nim
type
  IntArray = array[0..5, int] # an array that is indexed with 0..5
  IntSeq = seq[int] # a sequence of integers
var
  x: IntArray
  y: IntSeq
x = [1, 2, 3, 4, 5, 6]  # [] is the array constructor
y = @[1, 2, 3, 4, 5, 6] # the @ turns the array into a sequence

let z = [1.0, 2, 3, 4] # the type of z is array[0..3, float]
```

The lower bound of an array or sequence may be received by the built-in
proc `low()`, the higher bound by `high()`. The length may be received
by `len()`. `low()` for a sequence or an open array always returns 0, as
this is the first valid index. One can append elements to a sequence
with the `add()` proc or the `&` operator, and remove (and get) the last
element of a sequence with the `pop()` proc.

The notation `x[i]` can be used to access the i-th element of `x`.

Arrays are always bounds checked (statically or at runtime). These
checks can be disabled via pragmas or invoking the compiler with the
`--boundChecks:off`{.interpreted-text role="option"} command-line
switch.

An array constructor can have explicit indexes for readability:

``` nim
type
  Values = enum
    valA, valB, valC

const
  lookupTable = [
    valA: "A",
    valB: "B",
    valC: "C"
  ]
```

If an index is left out, `succ(lastIndex)` is used as the index value:

``` nim
type
  Values = enum
    valA, valB, valC, valD, valE

const
  lookupTable = [
    valA: "A",
    "B",
    valC: "C",
    "D", "e"
  ]
```

### Open arrays

Often fixed size arrays turn out to be too inflexible; procedures should
be able to deal with arrays of different sizes. The
`openarray`{.interpreted-text role="idx"} type allows this; it can only
be used for parameters. Openarrays are always indexed with an `int`
starting at position 0. The `len`, `low` and `high` operations are
available for open arrays too. Any array with a compatible base type can
be passed to an openarray parameter, the index type does not matter. In
addition to arrays, sequences can also be passed to an open array
parameter.

The openarray type cannot be nested: multidimensional openarrays are not
supported because this is seldom needed and cannot be done efficiently.

``` nim
proc testOpenArray(x: openArray[int]) = echo repr(x)
testOpenArray([1,2,3])  # array[]
testOpenArray(@[1,2,3]) # seq[]
```

### Varargs

A `varargs` parameter is an openarray parameter that additionally allows
to pass a variable number of arguments to a procedure. The compiler
converts the list of arguments to an array implicitly:

``` nim
proc myWriteln(f: File, a: varargs[string]) =
for s in items(a):
write(f, s)
write(f, "\n")
myWriteln(stdout, "abc", "def", "xyz")
# is transformed to:
myWriteln(stdout, ["abc", "def", "xyz"])
```

This transformation is only done if the varargs parameter is the last
parameter in the procedure header. It is also possible to perform type
conversions in this context:

``` nim
proc myWriteln(f: File, a: varargs[string, `$`]) =
for s in items(a):
write(f, s)
write(f, "\n")
myWriteln(stdout, 123, "abc", 4.0)
# is transformed to:
myWriteln(stdout, [$123, $"abc", $4.0])
```

In this example `$` is applied to any argument that is passed to the
parameter `a`. (Note that `$` applied to strings is a nop.)

Note that an explicit array constructor passed to a `varargs` parameter
is not wrapped in another implicit array construction:

``` nim
proc takeV[T](a: varargs[T]) = discard
takeV([123, 2, 1]) # takeV's T is "int", not "array of int"
```

`varargs[typed]` is treated specially: It matches a variable list of
arguments of arbitrary type but *always* constructs an implicit array.
This is required so that the builtin `echo` proc does what is expected:

``` nim
proc echo*(x: varargs[typed, `$`]) {...}
echo @[1, 2, 3]
# prints "@[1, 2, 3]" and not "123"
```

### Unchecked arrays

The `UncheckedArray[T]` type is a special kind of `array` where its
bounds are not checked. This is often useful to implement customized
flexibly sized arrays. Additionally, an unchecked array is translated
into a C array of undetermined size:

``` nim
type
MySeq = object
len, cap: int
data: UncheckedArray[int]
```

Produces roughly this C code:

``` C
typedef struct {
NI len;
NI cap;
NI data[];
} MySeq;
```

The base type of the unchecked array may not contain any GC\'ed memory
but this is currently not checked.

**Future directions**: GC\'ed memory should be allowed in unchecked
arrays and there should be an explicit annotation of how the GC is to
determine the runtime size of the array.

### Tuples and object types

A variable of a tuple or object type is a heterogeneous storage
container. A tuple or object defines various named *fields* of a type. A
tuple also defines a lexicographic *order* of the fields. Tuples are
meant to be heterogeneous storage types with few abstractions. The `()`
syntax can be used to construct tuples. The order of the fields in the
constructor must match the order of the tuple\'s definition. Different
tuple-types are *equivalent* if they specify the same fields of the same
type in the same order. The *names* of the fields also have to be the
same.

The assignment operator for tuples copies each component. The default
assignment operator for objects copies each component. Overloading of
the assignment operator is described
[here](manual_experimental.html#type-bound-operations).

``` nim
type
  Person = tuple[name: string, age: int] # type representing a person:
                                         # it consists of a name and an age.
var person: Person
person = (name: "Peter", age: 30)
assert person.name == "Peter"
# the same, but less readable:
person = ("Peter", 30)
assert person[0] == "Peter"
assert Person is (string, int)
assert (string, int) is Person
assert Person isnot tuple[other: string, age: int] # `other` is a different identifier
```

A tuple with one unnamed field can be constructed with the parentheses
and a trailing comma:

``` nim
proc echoUnaryTuple(a: (int,)) =
echo a[0]
echoUnaryTuple (1,)
```

In fact, a trailing comma is allowed for every tuple construction.

The implementation aligns the fields for the best access performance.
The alignment is compatible with the way the C compiler does it.

For consistency with `object` declarations, tuples in a `type` section
can also be defined with indentation instead of \`\[\]\`:

``` nim
type
Person = tuple   # type representing a person
name: string   # a person consists of a name
age: Natural   # and an age
```

Objects provide many features that tuples do not. Objects provide
inheritance and the ability to hide fields from other modules. Objects
with inheritance enabled have information about their type at runtime so
that the `of` operator can be used to determine the object\'s type. The
`of` operator is similar to the `instanceof` operator in Java.

``` nim
type
Person = object of RootObj
name*: string   # the * means that `name` is accessible from other modules
age: int        # no * means that the field is hidden
Student = ref object of Person # a student is a person
  id: int                      # with an id field

var
student: Student
person: Person
assert(student of Student) # is true
assert(student of Person) # also true
```

Object fields that should be visible from outside the defining module
have to be marked by `*`. In contrast to tuples, different object types
are never *equivalent*, they are nominal types whereas tuples are
structural. Objects that have no ancestor are implicitly `final` and
thus have no hidden type information. One can use the `inheritable`
pragma to introduce new object roots apart from `system.RootObj`.

``` nim
type
Person = object # example of a final object
name*: string
age: int
Student = ref object of Person # Error: inheritance only works with non-final objects
  id: int
```

### Object construction

Objects can also be created with an
`object construction expression`{.interpreted-text role="idx"} that has
the syntax `T(fieldA: valueA, fieldB: valueB, ...)` where `T` is an
`object` type or a `ref object` type:

``` nim
type
Student = object
name: string
age: int
PStudent = ref Student
var a1 = Student(name: "Anton", age: 5)
var a2 = PStudent(name: "Anton", age: 5)
# this also works directly:
var a3 = (ref Student)(name: "Anton", age: 5)
# not all fields need to be mentioned, and they can be mentioned out of order:
var a4 = Student(age: 5)
```

Note that, unlike tuples, objects require the field names along with
their values. For a `ref object` type `system.new` is invoked
implicitly.

### Object variants

Often an object hierarchy is an overkill in certain situations where
simple variant types are needed. Object variants are tagged unions
discriminated via an enumerated type used for runtime type flexibility,
mirroring the concepts of *sum types* and *algebraic data types (ADTs)*
as found in other languages.

An example:

``` nim
# This is an example of how an abstract syntax tree could be modelled in Nim
type
  NodeKind = enum  # the different node types
    nkInt,          # a leaf with an integer value
    nkFloat,        # a leaf with a float value
    nkString,       # a leaf with a string value
    nkAdd,          # an addition
    nkSub,          # a subtraction
    nkIf            # an if statement
  Node = ref NodeObj
  NodeObj = object
    case kind: NodeKind  # the `kind` field is the discriminator
    of nkInt: intVal: int
    of nkFloat: floatVal: float
    of nkString: strVal: string
    of nkAdd, nkSub:
      leftOp, rightOp: Node
    of nkIf:
      condition, thenPart, elsePart: Node

# create a new case object:
var n = Node(kind: nkIf, condition: nil)
# accessing n.thenPart is valid because the `nkIf` branch is active:
n.thenPart = Node(kind: nkFloat, floatVal: 2.0)

# the following statement raises an `FieldDefect` exception, because
# n.kind's value does not fit and the `nkString` branch is not active:
n.strVal = ""

# invalid: would change the active object branch:
n.kind = nkInt

var x = Node(kind: nkAdd, leftOp: Node(kind: nkInt, intVal: 4),
                          rightOp: Node(kind: nkInt, intVal: 2))
# valid: does not change the active object branch:
x.kind = nkSub
```

As can be seen from the example, an advantage to an object hierarchy is
that no casting between different object types is needed. Yet, access to
invalid object fields raises an exception.

The syntax of `case` in an object declaration follows closely the syntax
of the `case` statement: The branches in a `case` section may be
indented too.

In the example, the `kind` field is called the
`discriminator`{.interpreted-text role="idx"}: For safety, its address
cannot be taken and assignments to it are restricted: The new value must
not lead to a change of the active object branch. Also, when the fields
of a particular branch are specified during object construction, the
corresponding discriminator value must be specified as a constant
expression.

Instead of changing the active object branch, replace the old object in
memory with a new one completely:

``` nim
var x = Node(kind: nkAdd, leftOp: Node(kind: nkInt, intVal: 4),
                          rightOp: Node(kind: nkInt, intVal: 2))
# change the node's contents:
x[] = NodeObj(kind: nkString, strVal: "abc")
```

Starting with version 0.20 `system.reset` cannot be used anymore to
support object branch changes as this never was completely memory safe.

As a special rule, the discriminator kind can also be bounded using a
`case` statement. If possible values of the discriminator variable in a
`case` statement branch are a subset of discriminator values for the
selected object branch, the initialization is considered valid. This
analysis only works for immutable discriminators of an ordinal type and
disregards `elif` branches. For discriminator values with a `range`
type, the compiler checks if the entire range of possible values for the
discriminator value is valid for the chosen object branch.

A small example:

``` nim
let unknownKind = nkSub

# invalid: unsafe initialization because the kind field is not statically known:
var y = Node(kind: unknownKind, strVal: "y")

var z = Node()
case unknownKind
of nkAdd, nkSub:
  # valid: possible values of this branch are a subset of nkAdd/nkSub object branch:
  z = Node(kind: unknownKind, leftOp: Node(), rightOp: Node())
else:
  echo "ignoring: ", unknownKind

# also valid, since unknownKindBounded can only contain the values nkAdd or nkSub
let unknownKindBounded = range[nkAdd..nkSub](unknownKind)
z = Node(kind: unknownKindBounded, leftOp: Node(), rightOp: Node())
```

### cast uncheckedAssign

Some restrictions for case objects can be disabled via a
`{.cast(uncheckedAssign).}` section:

``` {.nim test="\"nim c $1\""}
type
  TokenKind* = enum
    strLit, intLit
  Token = object
    case kind*: TokenKind
    of strLit:
      s*: string
    of intLit:
      i*: int64

proc passToVar(x: var TokenKind) = discard

var t = Token(kind: strLit, s: "abc")

{.cast(uncheckedAssign).}:
  # inside the 'cast' section it is allowed to pass 't.kind' to a 'var T' parameter:
  passToVar(t.kind)

  # inside the 'cast' section it is allowed to set field 's' even though the
  # constructed 'kind' field has an unknown value:
  t = Token(kind: t.kind, s: "abc")

  # inside the 'cast' section it is allowed to assign to the 't.kind' field directly:
  t.kind = intLit
```

### Set type

### Reference and pointer types

References (similar to pointers in other programming languages) are a
way to introduce many-to-one relationships. This means different
references can point to and modify the same location in memory (also
called `aliasing`{.interpreted-text role="idx"}).

Nim distinguishes between `traced`{.interpreted-text role="idx"} and
`untraced`{.interpreted-text role="idx"} references. Untraced references
are also called *pointers*. Traced references point to objects of a
garbage-collected heap, untraced references point to manually allocated
objects or objects somewhere else in memory. Thus untraced references
are *unsafe*. However, for certain low-level operations (accessing the
hardware) untraced references are unavoidable.

Traced references are declared with the **ref** keyword, untraced
references are declared with the **ptr** keyword. In general, a `ptr T`
is implicitly convertible to the `pointer` type.

An empty subscript `[]` notation can be used to de-refer a reference,
the `addr` procedure returns the address of an item. An address is
always an untraced reference. Thus the usage of `addr` is an *unsafe*
feature.

The `.` (access a tuple/object field operator) and `[]`
(array/string/sequence index operator) operators perform implicit
dereferencing operations for reference types:

``` nim
type
  Node = ref NodeObj
  NodeObj = object
    le, ri: Node
    data: int

var
  n: Node
new(n)
n.data = 9
# no need to write n[].data; in fact n[].data is highly discouraged!
```

Automatic dereferencing can be performed for the first argument of a
routine call, but this is an experimental feature and is described
[here](manual_experimental.html#automatic-dereferencing).

In order to simplify structural type checking, recursive tuples are not
valid:

``` nim
# invalid recursion
type MyTuple = tuple[a: ref MyTuple]
```

Likewise `T = ref T` is an invalid type.

As a syntactical extension, `object` types can be anonymous if declared
in a type section via the `ref object` or `ptr object` notations. This
feature is useful if an object should only gain reference semantics:

``` nim
type
  Node = ref object
    le, ri: Node
    data: int
```

To allocate a new traced object, the built-in procedure `new` has to be
used. To deal with untraced memory, the procedures `alloc`, `dealloc`
and `realloc` can be used. The documentation of the
[system](system.html) module contains further information.

### Nil

If a reference points to *nothing*, it has the value `nil`. `nil` is the
default value for all `ref` and `ptr` types. The `nil` value can also be
used like any other literal value. For example, it can be used in an
assignment like `myRef = nil`.

Dereferencing `nil` is an unrecoverable fatal runtime error (and not a
panic).

A successful dereferencing operation `p[]` implies that `p` is not nil.
This can be exploited by the implementation to optimize code like:

``` nim
p[].field = 3
if p != nil:
  # if p were nil, `p[]` would have caused a crash already,
  # so we know `p` is always not nil here.
  action()
```

Into:

``` nim
p[].field = 3
action()
```

*Note*: This is not comparable to C\'s \"undefined behavior\" for
dereferencing NULL pointers.

### Mixing GC\'ed memory with `ptr`

Special care has to be taken if an untraced object contains traced
objects like traced references, strings, or sequences: in order to free
everything properly, the built-in procedure `reset` has to be called
before freeing the untraced memory manually:

``` nim
type
Data = tuple[x, y: int, s: string]
# allocate memory for Data on the heap:
var d = cast[ptr Data](alloc0(sizeof(Data)))

# create a new string on the garbage collected heap:
d.s = "abc"

# tell the GC that the string is not needed anymore:
reset(d.s)

# free the memory:
dealloc(d)
```

Without the `reset` call the memory allocated for the `d.s` string would
never be freed. The example also demonstrates two important features for
low-level programming: the `sizeof` proc returns the size of a type or
value in bytes. The `cast` operator can circumvent the type system: the
compiler is forced to treat the result of the `alloc0` call (which
returns an untyped pointer) as if it would have the type `ptr Data`.
Casting should only be done if it is unavoidable: it breaks type safety
and bugs can lead to mysterious crashes.

**Note**: The example only works because the memory is initialized to
zero (`alloc0` instead of `alloc` does this): `d.s` is thus initialized
to binary zero which the string assignment can handle. One needs to know
low-level details like this when mixing garbage-collected data with
unmanaged memory.

### Procedural type

A procedural type is internally a pointer to a procedure. `nil` is an
allowed value for a variable of a procedural type.

Examples:

``` nim
proc printItem(x: int) = ...

proc forEach(c: proc (x: int) {.cdecl.}) =
  ...

forEach(printItem)  # this will NOT compile because calling conventions differ
```

``` nim
type
  OnMouseMove = proc (x, y: int) {.closure.}

proc onMouseMove(mouseX, mouseY: int) =
  # has default calling convention
  echo "x: ", mouseX, " y: ", mouseY

proc setOnMouseMove(mouseMoveEvent: OnMouseMove) = discard

# ok, 'onMouseMove' has the default calling convention, which is compatible
# to 'closure':
setOnMouseMove(onMouseMove)
```

A subtle issue with procedural types is that the calling convention of
the procedure influences the type compatibility: procedural types are
only compatible if they have the same calling convention. As a special
extension, a procedure of the calling convention `nimcall` can be passed
to a parameter that expects a proc of the calling convention `closure`.

Nim supports these `calling conventions`{.interpreted-text role="idx"}:

`nimcall`{.interpreted-text role="idx"}

:   is the default convention used for a Nim **proc**. It is the same as
    `fastcall`, but only for C compilers that support `fastcall`.

`closure`{.interpreted-text role="idx"}

:   is the default calling convention for a **procedural type** that
    lacks any pragma annotations. It indicates that the procedure has a
    hidden implicit parameter (an *environment*). Proc vars that have
    the calling convention `closure` take up two machine words: One for
    the proc pointer and another one for the pointer to implicitly
    passed environment.

`stdcall`{.interpreted-text role="idx"}

:   This is the stdcall convention as specified by Microsoft. The
    generated C procedure is declared with the `__stdcall` keyword.

`cdecl`{.interpreted-text role="idx"}

:   The cdecl convention means that a procedure shall use the same
    convention as the C compiler. Under Windows the generated C
    procedure is declared with the `__cdecl` keyword.

`safecall`{.interpreted-text role="idx"}

:   This is the safecall convention as specified by Microsoft. The
    generated C procedure is declared with the `__safecall` keyword. The
    word *safe* refers to the fact that all hardware registers shall be
    pushed to the hardware stack.

`inline`{.interpreted-text role="idx"}

:   The inline convention means the caller should not call the
    procedure, but inline its code directly. Note that Nim does not
    inline, but leaves this to the C compiler; it generates `__inline`
    procedures. This is only a hint for the compiler: it may completely
    ignore it and it may inline procedures that are not marked as
    `inline`.

`fastcall`{.interpreted-text role="idx"}

:   Fastcall means different things to different C compilers. One gets
    whatever the C `__fastcall` means.

`thiscall`{.interpreted-text role="idx"}

:   This is the thiscall calling convention as specified by Microsoft,
    used on C++ class member functions on the x86 architecture.

`syscall`{.interpreted-text role="idx"}

:   The syscall convention is the same as `__syscall`{.interpreted-text
    role="c"} in C. It is used for interrupts.

`noconv`{.interpreted-text role="idx"}

:   The generated C code will not have any explicit calling convention
    and thus use the C compiler\'s default calling convention. This is
    needed because Nim\'s default calling convention for procedures is
    `fastcall` to improve speed.

Most calling conventions exist only for the Windows 32-bit platform.

The default calling convention is `nimcall`, unless it is an inner proc
(a proc inside of a proc). For an inner proc an analysis is performed
whether it accesses its environment. If it does so, it has the calling
convention `closure`, otherwise it has the calling convention `nimcall`.

### Distinct type

A `distinct` type is a new type derived from a
`base type`{.interpreted-text role="idx"} that is incompatible with its
base type. In particular, it is an essential property of a distinct type
that it **does not** imply a subtype relation between it and its base
type. Explicit type conversions from a distinct type to its base type
and vice versa are allowed. See also `distinctBase` to get the reverse
operation.

A distinct type is an ordinal type if its base type is an ordinal type.

#### Modeling currencies

A distinct type can be used to model different physical
`units`{.interpreted-text role="idx"} with a numerical base type, for
example. The following example models currencies.

Different currencies should not be mixed in monetary calculations.
Distinct types are a perfect tool to model different currencies:

``` nim
type
Dollar = distinct int
Euro = distinct int
var
  d: Dollar
  e: Euro

echo d + 12
# Error: cannot add a number with no unit and a `Dollar`
```

Unfortunately, `d + 12.Dollar` is not allowed either, because `+` is
defined for `int` (among others), not for `Dollar`. So a `+` for dollars
needs to be defined:

``` proc `+` (x, y: Dollar): Dollar =
result = Dollar(int(x) + int(y))
```

It does not make sense to multiply a dollar with a dollar, but with a
number without unit; and the same holds for division:

``` proc `*` (x: Dollar, y: int): Dollar =
result = Dollar(int(x) * y)
proc `*` (x: int, y: Dollar): Dollar =
  result = Dollar(x * int(y))

proc `div` ...
```

This quickly gets tedious. The implementations are trivial and the
compiler should not generate all this code only to optimize it away
later - after all `+` for dollars should produce the same binary code as
`+` for ints. The pragma `borrow`{.interpreted-text role="idx"} has been
designed to solve this problem; in principle, it generates the above
trivial implementations:

``` nim
proc `*` (x: Dollar, y: int): Dollar {.borrow.}
proc `*` (x: int, y: Dollar): Dollar {.borrow.}
proc `div` (x: Dollar, y: int): Dollar {.borrow.}
```

The `borrow` pragma makes the compiler use the same implementation as
the proc that deals with the distinct type\'s base type, so no code is
generated.

But it seems all this boilerplate code needs to be repeated for the
`Euro` currency. This can be solved with [templates](#templates).

``` {.nim test="\"nim c $1\""}
template additive(typ: typedesc) =
  proc `+` *(x, y: typ): typ {.borrow.}
  proc `-` *(x, y: typ): typ {.borrow.}

  # unary operators:
  proc `+` *(x: typ): typ {.borrow.}
  proc `-` *(x: typ): typ {.borrow.}

template multiplicative(typ, base: typedesc) =
  proc `*` *(x: typ, y: base): typ {.borrow.}
  proc `*` *(x: base, y: typ): typ {.borrow.}
  proc `div` *(x: typ, y: base): typ {.borrow.}
  proc `mod` *(x: typ, y: base): typ {.borrow.}

template comparable(typ: typedesc) =
  proc `<` * (x, y: typ): bool {.borrow.}
  proc `<=` * (x, y: typ): bool {.borrow.}
  proc `==` * (x, y: typ): bool {.borrow.}

template defineCurrency(typ, base: untyped) =
  type
    typ* = distinct base
  additive(typ)
  multiplicative(typ, base)
  comparable(typ)

defineCurrency(Dollar, int)
defineCurrency(Euro, int)
```

The borrow pragma can also be used to annotate the distinct type to
allow certain builtin operations to be lifted:

``` nim
type
Foo = object
a, b: int
s: string
Bar {.borrow: `.`.} = distinct Foo

var bb: ref Bar
new bb
# field access now valid
bb.a = 90
bb.s = "abc"
```

Currently, only the dot accessor can be borrowed in this way.

#### Avoiding SQL injection attacks

An SQL statement that is passed from Nim to an SQL database might be
modeled as a string. However, using string templates and filling in the
values is vulnerable to the famous
`SQL injection attack`{.interpreted-text role="idx"}:

``` nim
import std/strutils
proc query(db: DbHandle, statement: string) = ...

var
  username: string

db.query("SELECT FROM users WHERE name = '$1'" % username)
# Horrible security hole, but the compiler does not mind!
```

This can be avoided by distinguishing strings that contain SQL from
strings that don\'t. Distinct types provide a means to introduce a new
string type `SQL` that is incompatible with \`string\`:

``` nim
type
SQL = distinct string
proc query(db: DbHandle, statement: SQL) = ...

var
  username: string

db.query("SELECT FROM users WHERE name = '$1'" % username)
# Static error: `query` expects an SQL string!
```

It is an essential property of abstract types that they **do not** imply
a subtype relation between the abstract type and its base type. Explicit
type conversions from `string` to `SQL` are allowed:

``` nim
import std/[strutils, sequtils]
proc properQuote(s: string): SQL =
  # quotes a string properly for an SQL statement
  return SQL(s)

proc `%` (frmt: SQL, values: openarray[string]): SQL =
  # quote each argument:
  let v = values.mapIt(properQuote(it))
  # we need a temporary type for the type conversion :-(
  type StrSeq = seq[string]
  # call strutils.`%`:
  result = SQL(string(frmt) % StrSeq(v))

db.query("SELECT FROM users WHERE name = '$1'".SQL % [username])
```

Now we have compile-time checking against SQL injection attacks. Since
`"".SQL` is transformed to `SQL("")` no new syntax is needed for nice
looking `SQL` string literals. The hypothetical `SQL` type actually
exists in the library as the [SqlQuery type](db_common.html#SqlQuery) of
modules like [db_sqlite](db_sqlite.html).

### Auto type

The `auto` type can only be used for return types and parameters. For
return types it causes the compiler to infer the type from the routine
body:

``` nim
proc returnsInt(): auto = 1984
```

For parameters it currently creates implicitly generic routines:

``` nim
proc foo(a, b: auto) = discard
```

Is the same as:

``` nim
proc foo[T1, T2](a: T1, b: T2) = discard
```

However, later versions of the language might change this to mean
\"infer the parameters\' types from the body\". Then the above `foo`
would be rejected as the parameters\' types can not be inferred from an
empty `discard` statement.

