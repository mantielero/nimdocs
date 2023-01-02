### Dynlib pragma for export

With the `dynlib` pragma, a procedure can also be exported to a dynamic
library. The pragma then has no argument and has to be used in
conjunction with the `exportc` pragma:

``` Nim
proc exportme(): int {.cdecl, exportc, dynlib.}
```

This is only useful if the program is compiled as a dynamic library via
the `--app:lib`{.interpreted-text role="option"} command-line option.

