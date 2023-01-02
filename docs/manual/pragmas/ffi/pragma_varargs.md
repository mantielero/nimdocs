### Varargs pragma

The `varargs` pragma can be applied to procedures only (and procedure
types). It tells Nim that the proc can take a variable number of
parameters after the last specified parameter. Nim string values will be
converted to C strings automatically:

``` Nim
proc printf(formatstr: cstring) {.nodecl, varargs.}
printf("hallo %s", "world") # "world" will be passed as C string
```

