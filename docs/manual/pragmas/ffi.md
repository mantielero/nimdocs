## Foreign function interface

Nim\'s `FFI` (foreign function interface)
is extensive and only the parts that scale to other future backends
(like the LLVM/JavaScript backends) are documented here.

### Importc pragma

The `importc` pragma provides a means to import a proc or a variable
from C. The optional argument is a string containing the C identifier.
If the argument is missing, the C name is the Nim identifier *exactly as
spelled*:

``` proc printf(formatstr: cstring) {.header: "<stdio.h>", importc: "printf", varargs.}
```

When `importc` is applied to a `let` statement it can omit its value
which will then be expected to come from C. This can be used to import a
C `const`{.interpreted-text role="c"}:

``` {.emit: "const int cconst = 42;".}
let cconst {.importc, nodecl.}: cint

assert cconst == 42
```

Note that this pragma has been abused in the past to also work in the JS
backend for JS objects and functions. Other backends do provide the same
feature under the same name. Also, when the target language is not set
to C, other pragmas are available:

> -   [importcpp](manual.html#implementation-specific-pragmas-importcpp-pragma)
> -   [importobjc](manual.html#implementation-specific-pragmas-importobjc-pragma)
> -   [importjs](manual.html#implementation-specific-pragmas-importjs-pragma)

``` Nim
proc p(s: cstring) {.importc: "prefix$1".}
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.

### Exportc pragma

The `exportc` pragma provides a means to export a type, a variable, or a
procedure to C. Enums and constants can\'t be exported. The optional
argument is a string containing the C identifier. If the argument is
missing, the C name is the Nim identifier *exactly as spelled*:

``` Nim
proc callme(formatstr: cstring) {.exportc: "callMe", varargs.}
```

Note that this pragma is somewhat of a misnomer: Other backends do
provide the same feature under the same name.

The string literal passed to `exportc` can be a format string:

``` Nim
proc p(s: string) {.exportc: "prefix$1".} =
echo s
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.

If the symbol should also be exported to a dynamic library, the `dynlib`
pragma should be used in addition to the `exportc` pragma. See [Dynlib
pragma for
export](#foreign-function-interface-dynlib-pragma-for-export).

### Extern pragma

Like `exportc` or `importc`, the `extern` pragma affects name mangling.
The string literal passed to `extern` can be a format string:

``` Nim
proc p(s: string) {.extern: "prefix$1".} =
echo s
```

In the example, the external name of `p` is set to `prefixp`. Only `$1`
is available and a literal dollar sign must be written as `$$`.

### Bycopy pragma

The `bycopy` pragma can be applied to an object or tuple type and
instructs the compiler to pass the type by value to procs:

``` nim
type
Vector {.bycopy.} = object
x, y, z: float
```

### Byref pragma

The `byref` pragma can be applied to an object or tuple type and
instructs the compiler to pass the type by reference (hidden pointer) to
procs.

### Varargs pragma

The `varargs` pragma can be applied to procedures only (and procedure
types). It tells Nim that the proc can take a variable number of
parameters after the last specified parameter. Nim string values will be
converted to C strings automatically:

``` Nim
proc printf(formatstr: cstring) {.nodecl, varargs.}
printf("hallo %s", "world") # "world" will be passed as C string
```

### Union pragma

The `union` pragma can be applied to any `object` type. It means all of
the object\'s fields are overlaid in memory. This produces a
`union`{.interpreted-text role="c"} instead of a
`struct`{.interpreted-text role="c"} in the generated C/C++ code. The
object declaration then must not use inheritance or any GC\'ed memory
but this is currently not checked.

**Future directions**: GC\'ed memory should be allowed in unions and the
GC should scan unions conservatively.

### Packed pragma

The `packed` pragma can be applied to any `object` type. It ensures that
the fields of an object are packed back-to-back in memory. It is useful
to store packets or messages from/to network or hardware drivers, and
for interoperability with C. Combining packed pragma with inheritance is
not defined, and it should not be used with GC\'ed memory (ref\'s).

**Future directions**: Using GC\'ed memory in packed pragma will result
in a static error. Usage with inheritance should be defined and
documented.

### Dynlib pragma for import

With the `dynlib` pragma, a procedure or a variable can be imported from
a dynamic library (`.dll` files for Windows, `lib*.so` files for UNIX).
The non-optional argument has to be the name of the dynamic library:

``` Nim
proc gtk_image_new(): PGtkWidget
{.cdecl, dynlib: "libgtk-x11-2.0.so", importc.}
```

In general, importing a dynamic library does not require any special
linker options or linking with import libraries. This also implies that
no *devel* packages need to be installed.

The `dynlib` import mechanism supports a versioning scheme:

``` nim
proc Tcl_Eval(interp: pTcl_Interp, script: cstring): int {.cdecl,
importc, dynlib: "libtcl(|8.5|8.4|8.3).so.(1|0)".}
```

At runtime, the dynamic library is searched for (in this order):

    libtcl.so.1
    libtcl.so.0
    libtcl8.5.so.1
    libtcl8.5.so.0
    libtcl8.4.so.1
    libtcl8.4.so.0
    libtcl8.3.so.1
    libtcl8.3.so.0

The `dynlib` pragma supports not only constant strings as an argument
but also string expressions in general:

``` nim
import std/os
proc getDllName: string =
  result = "mylib.dll"
  if fileExists(result): return
  result = "mylib2.dll"
  if fileExists(result): return
  quit("could not load dynamic library")

proc myImport(s: cstring) {.cdecl, importc, dynlib: getDllName().}
```

**Note**: Patterns like `libtcl(|8.5|8.4).so` are only supported in
constant strings, because they are precompiled.

**Note**: Passing variables to the `dynlib` pragma will fail at runtime
because of order of initialization problems.

**Note**: A `dynlib` import can be overridden with the
`--dynlibOverride:name`{.interpreted-text role="option"} command-line
option. The [Compiler User Guide](nimc.html) contains further
information.

### Dynlib pragma for export

With the `dynlib` pragma, a procedure can also be exported to a dynamic
library. The pragma then has no argument and has to be used in
conjunction with the `exportc` pragma:

``` Nim
proc exportme(): int {.cdecl, exportc, dynlib.}
```

This is only useful if the program is compiled as a dynamic library via
the `--app:lib`{.interpreted-text role="option"} command-line option.

