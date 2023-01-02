## Generics

Generics are Nim\'s means to parametrize procs, iterators or types with
`type parameters`{.interpreted-text role="idx"}. Depending on the
context, the brackets are used either to introduce type parameters or to
instantiate a generic proc, iterator, or type.

The following example shows how a generic binary tree can be modeled:

``` {.nim test="\"nim c $1\""}
type
  BinaryTree*[T] = ref object # BinaryTree is a generic type with
                              # generic param `T`
    le, ri: BinaryTree[T]     # left and right subtrees; may be nil
    data: T                   # the data stored in a node

proc newNode*[T](data: T): BinaryTree[T] =
  # constructor for a node
  result = BinaryTree[T](le: nil, ri: nil, data: data)

proc add*[T](root: var BinaryTree[T], n: BinaryTree[T]) =
  # insert a node into the tree
  if root == nil:
    root = n
  else:
    var it = root
    while it != nil:
      # compare the data items; uses the generic `cmp` proc
      # that works for any type that has a `==` and `<` operator
      var c = cmp(it.data, n.data)
      if c < 0:
        if it.le == nil:
          it.le = n
          return
        it = it.le
      else:
        if it.ri == nil:
          it.ri = n
          return
        it = it.ri

proc add*[T](root: var BinaryTree[T], data: T) =
  # convenience proc:
  add(root, newNode(data))

iterator preorder*[T](root: BinaryTree[T]): T =
  # Preorder traversal of a binary tree.
  # This uses an explicit stack (which is more efficient than
  # a recursive iterator factory).
  var stack: seq[BinaryTree[T]] = @[root]
  while stack.len > 0:
    var n = stack.pop()
    while n != nil:
      yield n.data
      add(stack, n.ri)  # push right subtree onto the stack
      n = n.le          # and follow the left pointer

var
  root: BinaryTree[string] # instantiate a BinaryTree with `string`
add(root, newNode("hello")) # instantiates `newNode` and `add`
add(root, "world")          # instantiates the second `add` proc
for str in preorder(root):
  stdout.writeLine(str)
```

The `T` is called a `generic type parameter`{.interpreted-text
role="idx"} or a `type variable`{.interpreted-text role="idx"}.

### Is operator

The `is` operator is evaluated during semantic analysis to check for
type equivalence. It is therefore very useful for type specialization
within generic code:

``` nim
type
Table[Key, Value] = object
keys: seq[Key]
values: seq[Value]
when not (Key is string): # empty value for strings used for optimization
deletedKeys: seq[bool]
```

### Type Classes

A type class is a special pseudo-type that can be used to match against
types in the context of overload resolution or the `is` operator. Nim
supports the following built-in type classes:

  type class   matches
  ------------ -------------------
  `object`     any object type
  `tuple`      any tuple type
  `enum`       any enumeration
  `proc`       any proc type
  `ref`        any `ref` type
  `ptr`        any `ptr` type
  `var`        any `var` type
  `distinct`   any distinct type
  `array`      any array type
  `set`        any set type
  `seq`        any seq type
  `auto`       any type

Furthermore, every generic type automatically creates a type class of
the same name that will match any instantiation of the generic type.

Type classes can be combined using the standard boolean operators to
form more complex type classes:

``` nim
# create a type class that will match all tuple and object types
type RecordType = tuple or object
proc printFields[T: RecordType](rec: T) =
  for key, value in fieldPairs(rec):
    echo key, " = ", value
```

Type constraints on generic parameters can be grouped with `,` and
propagation stops with `;`, similarly to parameters for macros and
templates:

``` nim
proc fn1[T; U, V: SomeFloat]() = discard # T is unconstrained
template fn2(t; u, v: SomeFloat) = discard # t is unconstrained
```

Whilst the syntax of type classes appears to resemble that of
ADTs/algebraic data types in ML-like languages, it should be understood
that type classes are static constraints to be enforced at type
instantiations. Type classes are not really types in themselves but are
instead a system of providing generic \"checks\" that ultimately
*resolve* to some singular type. Type classes do not allow for runtime
type dynamism, unlike object variants or methods.

As an example, the following would not compile:

``` nim
type TypeClass = int | string
var foo: TypeClass = 2 # foo's type is resolved to an int here
foo = "this will fail" # error here, because foo is an int
```

Nim allows for type classes and regular types to be specified as
`type constraints`{.interpreted-text role="idx"} of the generic type
parameter:

``` nim
proc onlyIntOrString[T: int|string](x, y: T) = discard
onlyIntOrString(450, 616) # valid
onlyIntOrString(5.0, 0.0) # type mismatch
onlyIntOrString("xy", 50) # invalid as 'T' cannot be both at the same time
```

### Implicit generics

A type class can be used directly as the parameter\'s type.

``` nim
# create a type class that will match all tuple and object types
type RecordType = tuple or object

proc printFields(rec: RecordType) =
  for key, value in fieldPairs(rec):
    echo key, " = ", value
```

Procedures utilizing type classes in such a manner are considered to be
`implicitly generic`{.interpreted-text role="idx"}. They will be
instantiated once for each unique combination of param types used within
the program.

By default, during overload resolution, each named type class will bind
to exactly one concrete type. We call such type classes
`bind once`{.interpreted-text role="idx"} types. Here is an example
taken directly from the system module to illustrate this:

``` nim
proc `==`*(x, y: tuple): bool =
## requires `x` and `y` to be of the same tuple type
## generic `==` operator for tuples that is lifted from the components
## of `x` and `y`.
result = true
for a, b in fields(x, y):
if a != b: result = false
```

Alternatively, the `distinct` type modifier can be applied to the type
class to allow each param matching the type class to bind to a different
type. Such type classes are called `bind many`{.interpreted-text
role="idx"} types.

Procs written with the implicitly generic style will often need to refer
to the type parameters of the matched generic type. They can be easily
accessed using the dot syntax:

``` nim
type Matrix[T, Rows, Columns] = object
...
proc `[]`(m: Matrix, row, col: int): Matrix.T =
  m.data[col * high(Matrix.Columns) + row]
```

Here are more examples that illustrate implicit generics:

``` nim
proc p(t: Table; k: Table.Key): Table.Value

# is roughly the same as:

proc p[Key, Value](t: Table[Key, Value]; k: Key): Value
```

``` nim
proc p(a: Table, b: Table)

# is roughly the same as:

proc p[Key, Value](a, b: Table[Key, Value])
```

``` nim
proc p(a: Table, b: distinct Table)

# is roughly the same as:

proc p[Key, Value, KeyB, ValueB](a: Table[Key, Value], b: Table[KeyB, ValueB])
```

`typedesc` used as a parameter type also introduces an implicit generic.
`typedesc` has its own set of rules:

``` nim
proc p(a: typedesc)

# is roughly the same as:

proc p[T](a: typedesc[T])
```

`typedesc` is a \"bind many\" type class:

``` nim
proc p(a, b: typedesc)

# is roughly the same as:

proc p[T, T2](a: typedesc[T], b: typedesc[T2])
```

A parameter of type `typedesc` is itself usable as a type. If it is used
as a type, it\'s the underlying type. (In other words, one level of
\"typedesc\"-ness is stripped off:

``` nim
proc p(a: typedesc; b: a) = discard

# is roughly the same as:
proc p[T](a: typedesc[T]; b: T) = discard

# hence this is a valid call:
p(int, 4)
# as parameter 'a' requires a type, but 'b' requires a value.
```

### Generic inference restrictions

The types `var T` and `typedesc[T]` cannot be inferred in a generic
instantiation. The following is not allowed:

``` {.nim test="\"nim c $1\"" status="1"}
proc g[T](f: proc(x: T); x: T) =
  f(x)

proc c(y: int) = echo y
proc v(y: var int) =
  y += 100
var i: int

# allowed: infers 'T' to be of type 'int'
g(c, 42)

# not valid: 'T' is not inferred to be of type 'var int'
g(v, i)

# also not allowed: explicit instantiation via 'var int'
g[var int](v, i)
```

### Symbol lookup in generics

#### Open and Closed symbols

The symbol binding rules in generics are slightly subtle: There are
\"open\" and \"closed\" symbols. A \"closed\" symbol cannot be re-bound
in the instantiation context, an \"open\" symbol can. Per default,
overloaded symbols are open and every other symbol is closed.

Open symbols are looked up in two different contexts: Both the context
at definition and the context at instantiation are considered:

``` {.nim test="\"nim c $1\""}
type
  Index = distinct int

proc `==` (a, b: Index): bool {.borrow.}

var a = (0, 0.Index)
var b = (0, 0.Index)

echo a == b # works!
```

In the example, the generic `==` for tuples (as defined in the system
module) uses the `==` operators of the tuple\'s components. However, the
`==` for the `Index` type is defined *after* the `==` for tuples; yet
the example compiles as the instantiation takes the currently defined
symbols into account too.

### Mixin statement

A symbol can be forced to be open by a `mixin`{.interpreted-text
role="idx"} declaration:

``` {.nim test="\"nim c $1\""}
proc create*[T](): ref T =
  # there is no overloaded 'init' here, so we need to state that it's an
  # open symbol explicitly:
  mixin init
  new result
  init result
```

`mixin` statements only make sense in templates and generics.

### Bind statement

The `bind` statement is the counterpart to the `mixin` statement. It can
be used to explicitly declare identifiers that should be bound early
(i.e. the identifiers should be looked up in the scope of the
template/generic definition):

``` nim
# Module A
var
lastId = 0
template genId*: untyped =
  bind lastId
  inc(lastId)
  lastId
```

``` nim
# Module B
import A
echo genId()
```

But a `bind` is rarely useful because symbol binding from the definition
scope is the default.

`bind` statements only make sense in templates and generics.

### Delegating bind statements

The following example outlines a problem that can arise when generic
instantiations cross multiple different modules:

``` nim
# module A
proc genericA*[T](x: T) =
  mixin init
  init(x)
```

``` nim
import C

# module B
proc genericB*[T](x: T) =
  # Without the `bind init` statement C's init proc is
  # not available when `genericB` is instantiated:
  bind init
  genericA(x)
```

``` nim
# module C
type O = object
proc init*(x: var O) = discard
```

``` nim
# module main
import B, C

genericB O()
```

In module B has an `init` proc from module C in its scope that is not
taken into account when `genericB` is instantiated which leads to the
instantiation of `genericA`. The solution is to
`forward`{.interpreted-text role="idx"} these symbols by a `bind`
statement inside `genericB`.

