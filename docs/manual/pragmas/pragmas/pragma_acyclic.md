### acyclic pragma

The `acyclic` pragma can be used for object types to mark them as
acyclic even though they seem to be cyclic. This is an **optimization**
for the garbage collector to not consider objects of this type as part
of a cycle:

```nim
type
  Node = ref NodeObj
  NodeObj {.acyclic.} = object
    left, right: Node
    data: string
```

Or if we directly use a ref object:

```nim
type
  Node {.acyclic.} = ref object
    left, right: Node
    data: string
```

In the example, a tree structure is declared with the `Node` type. Note
that the type definition is recursive and the GC has to assume that
objects of this type may form a cyclic graph. The `acyclic` pragma
passes the information that this cannot happen to the GC. If the
programmer uses the `acyclic` pragma for data types that are in reality
cyclic, this may result in memory leaks, but memory safety is preserved.

