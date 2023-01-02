### line pragma

The `line` pragma can be used to affect line information of the
annotated statement, as seen in stack backtraces:

```nim
template myassert*(cond: untyped, msg = "") =
  if not cond:
    # change run-time line information of the 'raise' statement:
    {.line: instantiationInfo().}:
      raise newException(AssertionDefect, msg)
```

If the `line` pragma is used with a parameter, the parameter needs be a
`tuple[filename: string, line: int]`. If it is used without a parameter,
`system.instantiationInfo()` is used.

