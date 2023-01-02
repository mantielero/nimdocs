### shallow pragma

The `shallow` pragma affects the semantics of a type: The compiler is
allowed to make a shallow copy. This can cause serious semantic issues
and break memory safety! However, it can speed up assignments
considerably, because the semantics of Nim require deep copying of
sequences and strings. This can be expensive, especially if sequences
are used to build a tree structure:

```nim
type
  NodeKind = enum nkLeaf, nkInner
  Node {.shallow.} = object
    case kind: NodeKind
    of nkLeaf:
      strVal: string
    of nkInner:
      children: seq[Node]
```

