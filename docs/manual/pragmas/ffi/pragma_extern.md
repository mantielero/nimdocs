### Extern pragma

Like `exportc` or `importc`, the `extern` pragma affects name mangling.
The string literal passed to `extern` can be a format string:

```nim
proc p(s: string) {.extern: "prefix$1".} =
echo s
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.


