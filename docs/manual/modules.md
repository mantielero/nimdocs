## Modules

Nim supports splitting a program into pieces by a module concept. Each
module needs to be in its own file and has its own
`namespace`{.interpreted-text role="idx"}. Modules enable
`information hiding`{.interpreted-text role="idx"} and
`separate compilation`{.interpreted-text role="idx"}. A module may gain
access to symbols of another module by the `import`{.interpreted-text
role="idx"} statement. `Recursive module dependencies`{.interpreted-text
role="idx"} are allowed, but are slightly subtle. Only top-level symbols
that are marked with an asterisk (`*`) are exported. A valid module name
can only be a valid Nim identifier (and thus its filename is
`identifier.nim`).

The algorithm for compiling modules is:

-   Compile the whole module as usual, following import statements
    recursively.
-   If there is a cycle, only import the already parsed symbols (that
    are exported); if an unknown identifier occurs then abort.

This is best illustrated by an example:

``` nim
# Module A
type
T1* = int  # Module A exports the type `T1`
import B     # the compiler starts parsing B
proc main() =
  var i = p(3) # works because B has been parsed completely here

main()
```

``` nim
# Module B
import A  # A is not parsed here! Only the already known symbols
# of A are imported.
proc p*(x: A.T1): A.T1 =
  # this works because the compiler has already
  # added T1 to A's interface symbol table
  result = x + 1
```

### Import statement

After the `import` statement, a list of module names can follow or a
single module name followed by an `except` list to prevent some symbols
from being imported:

``` {.nim test="\"nim c $1\"" status="1"}
import std/strutils except `%`, toUpperAscii

# doesn't work then:
echo "$1" % "abc".toUpperAscii
```

It is not checked that the `except` list is really exported from the
module. This feature allows us to compile against an older version of
the module that does not export these identifiers.

The `import` statement is only allowed at the top level.

### Include statement

The `include` statement does something fundamentally different than
importing a module: it merely includes the contents of a file. The
`include` statement is useful to split up a large module into several
files:

``` nim
include fileA, fileB, fileC
```

The `include` statement can be used outside of the top level, as such:

``` nim
# Module A
echo "Hello World!"
```

``` nim
# Module B
proc main() =
include A
main() # => Hello World!
```

### Module names in imports

A module alias can be introduced via the `as` keyword:

``` nim
import std/strutils as su, std/sequtils as qu
echo su.format("$1", "lalelu")
```

The original module name is then not accessible. The notations
`path/to/module` or `"path/to/module"` can be used to refer to a module
in subdirectories:

``` nim
import lib/pure/os, "lib/pure/times"
```

Note that the module name is still `strutils` and not
`lib/pure/strutils` and so one **cannot** do:

``` nim
import lib/pure/strutils
echo lib/pure/strutils.toUpperAscii("abc")
```

Likewise, the following does not make sense as the name is `strutils`
already:

``` nim
import lib/pure/strutils as strutils
```

### Collective imports from a directory

The syntax `import dir / [moduleA, moduleB]` can be used to import
multiple modules from the same directory.

Path names are syntactically either Nim identifiers or string literals.
If the path name is not a valid Nim identifier it needs to be a string
literal:

``` nim
import "gfx/3d/somemodule" # in quotes because '3d' is not a valid Nim identifier
```

### Pseudo import/include paths

A directory can also be a so-called \"pseudo directory\". They can be
used to avoid ambiguity when there are multiple modules with the same
path.

There are two pseudo directories:

1.  \`std\`: The `std` pseudo directory is the abstract location of
    Nim\'s standard library. For example, the syntax
    `import std / strutils` is used to unambiguously refer to the
    standard library\'s `strutils` module.
2.  \`pkg\`: The `pkg` pseudo directory is used to unambiguously refer
    to a Nimble package. However, for technical details that lie outside
    the scope of this document, its semantics are: *Use the search path
    to look for module name but ignore the standard library locations*.
    In other words, it is the opposite of `std`.

It is recommended and preferred but not currently enforced that all
stdlib module imports include the std/ \"pseudo directory\" as part of
the import name.

### From import statement

After the `from` statement, a module name follows followed by an
`import` to list the symbols one likes to use without explicit full
qualification:

``` {.nim test="\"nim c $1\""}
from std/strutils import `%`

echo "$1" % "abc"
# always possible: full qualification:
echo strutils.replace("abc", "a", "z")
```

It\'s also possible to use `from module import nil` if one wants to
import the module but wants to enforce fully qualified access to every
symbol in `module`.

### Export statement

An `export` statement can be used for symbol forwarding so that client
modules don\'t need to import a module\'s dependencies:

``` nim
# module B
type MyObject* = object
```

``` nim
# module A
import B
export B.MyObject
proc `$`*(x: MyObject): string = "my object"
```

``` nim
# module C
import A
# B.MyObject has been imported implicitly here:
var x: MyObject
echo $x
```

When the exported symbol is another module, all of its definitions will
be forwarded. One can use an `except` list to exclude some of the
symbols.

Notice that when exporting, one needs to specify only the module name:

``` nim
import foo/bar/baz
export baz
```

### Scope rules

Identifiers are valid from the point of their declaration until the end
of the block in which the declaration occurred. The range where the
identifier is known is the scope of the identifier. The exact scope of
an identifier depends on the way it was declared.

#### Block scope

The *scope* of a variable declared in the declaration part of a block is
valid from the point of declaration until the end of the block. If a
block contains a second block, in which the identifier is redeclared,
then inside this block, the second declaration will be valid. Upon
leaving the inner block, the first declaration is valid again. An
identifier cannot be redefined in the same block, except if valid for
procedure or iterator overloading purposes.

#### Tuple or object scope

The field identifiers inside a tuple or object definition are valid in
the following places:

-   To the end of the tuple/object definition.
-   Field designators of a variable of the given tuple/object type.
-   In all descendant types of the object type.

#### Module scope

All identifiers of a module are valid from the point of declaration
until the end of the module. Identifiers from indirectly dependent
modules are *not* available. The `system`{.interpreted-text role="idx"}
module is automatically imported in every module.

If a module imports an identifier by two different modules, each
occurrence of the identifier has to be qualified unless it is an
overloaded procedure or iterator in which case the overloading
resolution takes place:

``` nim
# Module A
var x*: string
```

``` nim
# Module B
var x*: int
```

``` nim
# Module C
import A, B
write(stdout, x) # error: x is ambiguous
write(stdout, A.x) # no error: qualifier used
var x = 4
write(stdout, x) # not ambiguous: uses the module C's x
```

### Packages

A collection of modules in a file tree with an `identifier.nimble` file
in the root of the tree is called a Nimble package. A valid package name
can only be a valid Nim identifier and thus its filename is
`identifier.nimble` where `identifier` is the desired package name. A
module without a `.nimble` file is assigned the package identifier:
`unknown`.

The distinction between packages allows diagnostic compiler messages to
be scoped to the current project\'s package vs foreign packages.

