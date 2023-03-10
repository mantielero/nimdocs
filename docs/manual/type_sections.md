## Type sections

Example:

``` nim
type # example demonstrating mutually recursive types
Node = ref object  # an object managed by the garbage collector (ref)
le, ri: Node     # left and right subtrees
sym: ref Sym     # leaves contain a reference to a Sym
Sym = object       # a symbol
  name: string     # the symbol's name
  line: int        # the line the symbol was declared in
  code: Node       # the symbol's abstract syntax tree
```

A type section begins with the `type` keyword. It contains multiple type
definitions. A type definition binds a type to a name. Type definitions
can be recursive or even mutually recursive. Mutually recursive types
are only possible within a single `type` section. Nominal types like
`objects` or `enums` can only be defined in a `type` section.

