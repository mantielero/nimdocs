### Importc pragma
The `importc` pragma provides a means to import a proc or a variable
from C. The optional argument is a string containing the C identifier.
If the argument is missing, the C name is the Nim identifier *exactly as
spelled*:

```nim 
proc printf(formatstr: cstring) {.header: "<stdio.h>", importc: "printf", varargs.}
```

When `importc` is applied to a `let` statement it can omit its value
which will then be expected to come from C. This can be used to import a
C `const`{.interpreted-text role="c"}:

```nim 
{.emit: "const int cconst = 42;".}

let cconst {.importc, nodecl.}: cint

assert cconst == 42
```

Note that this pragma has been abused in the past to also work in the JS
backend for JS objects and functions. Other backends do provide the same
feature under the same name. Also, when the target language is not set
to C, other pragmas are available:

-   [importcpp](manual.html#implementation-specific-pragmas-importcpp-pragma)
-   [importobjc](manual.html#implementation-specific-pragmas-importobjc-pragma)
-   [importjs](manual.html#implementation-specific-pragmas-importjs-pragma)

```nim
proc p(s: cstring) {.importc: "prefix$1".}
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.

