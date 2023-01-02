## Converters

A converter is like an ordinary proc except that it enhances the
\"implicitly convertible\" type relation (see [Convertible
relation](#convertible-relation)):

``` nim
# bad style ahead: Nim is not C.
converter toBool(x: int): bool = x != 0
if 4:
  echo "compiles"
```

A converter can also be explicitly invoked for improved readability.
Note that implicit converter chaining is not supported: If there is a
converter from type A to type B and from type B to type C the implicit
conversion from A to C is not provided.
