## User-defined pragmas

### pragma pragma

The `pragma` pragma can be used to declare user-defined pragmas. This is
useful because Nim\'s templates and macros do not affect pragmas.
User-defined pragmas are in a different module-wide scope than all other
symbols. They cannot be imported from a module.

Example:

``` nim
when appType == "lib":
{.pragma: rtl, exportc, dynlib, cdecl.}
else:
{.pragma: rtl, importc, dynlib: "client.dll", cdecl.}
proc p*(a, b: int): int {.rtl.} =
  result = a+b
```

In the example, a new pragma named `rtl` is introduced that either
imports a symbol from a dynamic library or exports the symbol for
dynamic library generation.

### Custom annotations

It is possible to define custom typed pragmas. Custom pragmas do not
affect code generation directly, but their presence can be detected by
macros. Custom pragmas are defined using templates annotated with pragma
\`pragma\`:

``` nim
template dbTable(name: string, table_space: string = "") {.pragma.}
template dbKey(name: string = "", primary_key: bool = false) {.pragma.}
template dbForeignKey(t: typedesc) {.pragma.}
template dbIgnore {.pragma.}
```

Consider this stylized example of a possible Object Relation Mapping
(ORM) implementation:

``` nim
const tblspace {.strdefine.} = "dev" # switch for dev, test and prod environments
type
  User {.dbTable("users", tblspace).} = object
    id {.dbKey(primary_key = true).}: int
    name {.dbKey"full_name".}: string
    is_cached {.dbIgnore.}: bool
    age: int

  UserProfile {.dbTable("profiles", tblspace).} = object
    id {.dbKey(primary_key = true).}: int
    user_id {.dbForeignKey: User.}: int
    read_access: bool
    write_access: bool
    admin_acess: bool
```

In this example, custom pragmas are used to describe how Nim objects are
mapped to the schema of the relational database. Custom pragmas can have
zero or more arguments. In order to pass multiple arguments use one of
template call syntaxes. All arguments are typed and follow standard
overload resolution rules for templates. Therefore, it is possible to
have default values for arguments, pass by name, varargs, etc.

Custom pragmas can be used in all locations where ordinary pragmas can
be specified. It is possible to annotate procs, templates, type and
variable definitions, statements, etc.

The macros module includes helpers which can be used to simplify custom
pragma access `hasCustomPragma`, `getCustomPragmaVal`. Please consult
the [macros](macros.html) module documentation for details. These macros
are not magic, everything they do can also be achieved by walking the
AST of the object representation.

More examples with custom pragmas:

-   Better serialization/deserialization control:

    ``` nim
    type MyObj = object
    a {.dontSerialize.}: int
    b {.defaultDeserialize: 5.}: int
    c {.serializationKey: "_c".}: string
    ```

-   Adopting type for gui inspector in a game engine:

    ``` nim
    type MyComponent = object
    position {.editable, animatable.}: Vector3
    alpha {.editRange: [0.0..1.0], animatable.}: float32
    ```

### Macro pragmas

All macros and templates can also be used as pragmas. They can be
attached to routines (procs, iterators, etc), type names, or type
expressions. The compiler will perform the following simple syntactic
transformations:

``` nim
template command(name: string, def: untyped) = discard
proc p() {.command("print").} = discard
```

This is translated to:

``` nim
command("print"):
proc p() = discard
```

------------------------------------------------------------------------

``` nim
type
AsyncEventHandler = proc (x: Event) {.async.}
```

This is translated to:

``` nim
type
AsyncEventHandler = async(proc (x: Event))
```

------------------------------------------------------------------------

``` nim
type
MyObject {.schema: "schema.protobuf".} = object
```

This is translated to a call to the `schema` macro with a `nnkTypeDef`
AST node capturing both the left-hand side and right-hand side of the
definition. The macro can return a potentially modified `nnkTypeDef`
tree which will replace the original row in the type section.

When multiple macro pragmas are applied to the same definition, the
compiler will apply them consequently from left to right. Each macro
will receive as input the output of the previous one.