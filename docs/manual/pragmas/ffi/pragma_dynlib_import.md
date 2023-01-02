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

