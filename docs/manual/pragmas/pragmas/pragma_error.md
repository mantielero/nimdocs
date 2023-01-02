### error pragma

The `error` pragma is used to make the compiler output an error message
with the given content. The compilation does not necessarily abort after
an error though.

The `error` pragma can also be used to annotate a symbol (like an
iterator or proc). The *usage* of the symbol then triggers a static
error. This is especially useful to rule out that some operation is
valid due to overloading and type conversions:

```nim
## check that underlying int values are compared and not the pointers:
proc `==`(x, y: ptr int): bool {.error.}
```

