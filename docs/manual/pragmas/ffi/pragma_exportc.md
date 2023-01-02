### Exportc pragma

The `exportc` pragma provides a means to export a type, a variable, or a
procedure to C. Enums and constants can\'t be exported. The optional
argument is a string containing the C identifier. If the argument is
missing, the C name is the Nim identifier *exactly as spelled*:

```nim
proc callme(formatstr: cstring) {.exportc: "callMe", varargs.}
```

Note that this pragma is somewhat of a misnomer: Other backends do
provide the same feature under the same name.

The string literal passed to `exportc` can be a format string:

```nim
proc p(s: string) {.exportc: "prefix$1".} =
echo s
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.

If the symbol should also be exported to a dynamic library, the `dynlib`
pragma should be used in addition to the `exportc` pragma. See [Dynlib
pragma for export](#foreign-function-interface-dynlib-pragma-for-export).

