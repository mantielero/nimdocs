### global pragma

The `global` pragma can be applied to a variable within a proc to
instruct the compiler to store it in a global location and initialize it
once at program startup.

```nim
proc isHexNumber(s: string): bool =
var pattern {.global.} = re"[0-9a-fA-F]+"
result = s.match(pattern)
```

When used within a generic proc, a separate unique global variable will
be created for each instantiation of the proc. The order of
initialization of the created global variables within a module is not
defined, but all of them will be initialized after any top-level
variables in their originating module and before any variable in a
module that imports it.