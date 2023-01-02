### used pragma

Nim produces a warning for symbols that are not exported and not used
either. The `used` pragma can be attached to a symbol to suppress this
warning. This is particularly useful when the symbol was generated by a
macro:

```nim
template implementArithOps(T) =
proc echoAdd(a, b: T) {.used.} =
echo a + b
proc echoSub(a, b: T) {.used.} =
echo a - b
# no warning produced for the unused 'echoSub'
implementArithOps(int)
echoAdd 3, 5
```

`used` can also be used as a top-level statement to mark a module as
"used". This prevents the "Unused import" warning:

```nim
# module: debughelper.nim
when defined(nimHasUsed):
  # 'import debughelper' is so useful for debugging
  # that Nim shouldn't produce a warning for that import,
  # even if currently unused:
  {.used.}
```
