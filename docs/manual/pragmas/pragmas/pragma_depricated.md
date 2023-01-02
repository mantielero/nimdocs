### deprecated pragma

The deprecated pragma is used to mark a symbol as deprecated:

```nim
proc p() {.deprecated.}
var x {.deprecated.}: char
```

This pragma can also take in an optional warning string to relay to
developers.

```nim
proc thing(x: bool) {.deprecated: "use thong instead".}
```

