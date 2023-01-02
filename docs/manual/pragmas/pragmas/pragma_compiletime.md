### compileTime pragma

The `compileTime` pragma is used to mark a proc or variable to be used
only during compile-time execution. No code will be generated for it.
Compile-time procs are useful as helpers for macros. Since version
0.12.0 of the language, a proc that uses `system.NimNode` within its
parameter types is implicitly declared \`compileTime\`:

```nim
proc astHelper(n: NimNode): NimNode =
result = n
```

Is the same as:

```nim
proc astHelper(n: NimNode): NimNode {.compileTime.} =
result = n
```

`compileTime` variables are available at runtime too. This simplifies
certain idioms where variables are filled at compile-time (for example,
lookup tables) but accessed at runtime:

``` {.nim test=""nim c -r $1""}
import std/macros

var nameToProc {.compileTime.}: seq[(string, proc (): string {.nimcall.})]

macro registerProc(p: untyped): untyped =
  result = newTree(nnkStmtList, p)

  let procName = p[0]
  let procNameAsStr = $p[0]
  result.add quote do:
    nameToProc.add((`procNameAsStr`, `procName`))

proc foo: string {.registerProc.} = "foo"
proc bar: string {.registerProc.} = "bar"
proc baz: string {.registerProc.} = "baz"

doAssert nameToProc[2][1]() == "baz"
```

