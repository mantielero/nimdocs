## Macros

A macro is a special function that is executed at compile time.
Normally, the input for a macro is an abstract syntax tree (AST) of the
code that is passed to it. The macro can then do transformations on it
and return the transformed AST. This can be used to add custom language
features and implement `domain-specific languages`{.interpreted-text
role="idx"}.

Macro invocation is a case where semantic analysis does **not** entirely
proceed top to bottom and left to right. Instead, semantic analysis
happens at least twice:

-   Semantic analysis recognizes and resolves the macro invocation.
-   The compiler executes the macro body (which may invoke other procs).
-   It replaces the AST of the macro invocation with the AST returned by
    the macro.
-   It repeats semantic analysis of that region of the code.
-   If the AST returned by the macro contains other macro invocations,
    this process iterates.

While macros enable advanced compile-time code transformations, they
cannot change Nim\'s syntax.

### Debug Example

The following example implements a powerful `debug` command that accepts
a variable number of arguments:

``` {.nim test="\"nim c $1\""}
# to work with Nim syntax trees, we need an API that is defined in the
# `macros` module:
import std/macros

macro debug(args: varargs[untyped]): untyped =
  # `args` is a collection of `NimNode` values that each contain the
  # AST for an argument of the macro. A macro always has to
  # return a `NimNode`. A node of kind `nnkStmtList` is suitable for
  # this use case.
  result = nnkStmtList.newTree()
  # iterate over any argument that is passed to this macro:
  for n in args:
    # add a call to the statement list that writes the expression;
    # `toStrLit` converts an AST to its string representation:
    result.add newCall("write", newIdentNode("stdout"), newLit(n.repr))
    # add a call to the statement list that writes ": "
    result.add newCall("write", newIdentNode("stdout"), newLit(": "))
    # add a call to the statement list that writes the expressions value:
    result.add newCall("writeLine", newIdentNode("stdout"), n)

var
  a: array[0..10, int]
  x = "some string"
a[0] = 42
a[1] = 45

debug(a[0], a[1], x)
```

The macro call expands to:

``` nim
write(stdout, "a[0]")
write(stdout, ": ")
writeLine(stdout, a[0])
write(stdout, "a[1]")
write(stdout, ": ")
writeLine(stdout, a[1])

write(stdout, "x")
write(stdout, ": ")
writeLine(stdout, x)
```

Arguments that are passed to a `varargs` parameter are wrapped in an
array constructor expression. This is why `debug` iterates over all of
`args`\'s children.

### BindSym

The above `debug` macro relies on the fact that `write`, `writeLine` and
`stdout` are declared in the system module and are thus visible in the
instantiating context. There is a way to use bound identifiers (aka
`symbols`) instead of using unbound
identifiers. The `bindSym` builtin can be used for that:

``` {.nim test="\"nim c $1\""}
import std/macros

macro debug(n: varargs[typed]): untyped =
  result = newNimNode(nnkStmtList, n)
  for x in n:
    # we can bind symbols in scope via 'bindSym':
    add(result, newCall(bindSym"write", bindSym"stdout", toStrLit(x)))
    add(result, newCall(bindSym"write", bindSym"stdout", newStrLitNode(": ")))
    add(result, newCall(bindSym"writeLine", bindSym"stdout", x))

var
  a: array[0..10, int]
  x = "some string"
a[0] = 42
a[1] = 45

debug(a[0], a[1], x)
```

The macro call expands to:

``` nim
write(stdout, "a[0]")
write(stdout, ": ")
writeLine(stdout, a[0])
write(stdout, "a[1]")
write(stdout, ": ")
writeLine(stdout, a[1])

write(stdout, "x")
write(stdout, ": ")
writeLine(stdout, x)
```

However, the symbols `write`, `writeLine` and `stdout` are already bound
and are not looked up again. As the example shows, `bindSym` does work
with overloaded symbols implicitly.

### Case-Of Macro

In Nim, it is possible to have a macro with the syntax of a *case-of*
expression just with the difference that all *of-branches* are passed to
and processed by the macro implementation. It is then up the macro
implementation to transform the *of-branches* into a valid Nim
statement. The following example should show how this feature could be
used for a lexical analyzer.

``` nim
import std/macros
macro case_token(args: varargs[untyped]): untyped =
  echo args.treeRepr
  # creates a lexical analyzer from regular expressions
  # ... (implementation is an exercise for the reader ;-)
  discard

case_token: # this colon tells the parser it is a macro statement
of r"[A-Za-z_]+[A-Za-z_0-9]*":
  return tkIdentifier
of r"0-9+":
  return tkInteger
of r"[\+\-\*\?]+":
  return tkOperator
else:
  return tkUnknown
```

**Style note**: For code readability, it is best to use the least
powerful programming construct that still suffices. So the \"check
list\" is:

(1) Use an ordinary proc/iterator, if possible.
(2) Else: Use a generic proc/iterator, if possible.
(3) Else: Use a template, if possible.
(4) Else: Use a macro.

### For loop macro

A macro that takes as its only input parameter an expression of the
special type `system.ForLoopStmt` can rewrite the entirety of a `for`
loop:

``` {.nim test="\"nim c $1\""}
import std/macros

macro example(loop: ForLoopStmt) =
  result = newTree(nnkForStmt)    # Create a new For loop.
  result.add loop[^3]             # This is "item".
  result.add loop[^2][^1]         # This is "[1, 2, 3]".
  result.add newCall(bindSym"echo", loop[0])

for item in example([1, 2, 3]): discard
```

Expands to:

``` nim
for item in items([1, 2, 3]):
echo item
```

Another example:

``` {.nim test="\"nim c $1\""}
import std/macros

macro enumerate(x: ForLoopStmt): untyped =
  expectKind x, nnkForStmt
  # check if the starting count is specified:
  var countStart = if x[^2].len == 2: newLit(0) else: x[^2][1]
  result = newStmtList()
  # we strip off the first for loop variable and use it as an integer counter:
  result.add newVarStmt(x[0], countStart)
  var body = x[^1]
  if body.kind != nnkStmtList:
    body = newTree(nnkStmtList, body)
  body.add newCall(bindSym"inc", x[0])
  var newFor = newTree(nnkForStmt)
  for i in 1..x.len-3:
    newFor.add x[i]
  # transform enumerate(X) to 'X'
  newFor.add x[^2][^1]
  newFor.add body
  result.add newFor
  # now wrap the whole macro in a block to create a new scope
  result = quote do:
    block: `result`

for a, b in enumerate(items([1, 2, 3])):
  echo a, " ", b

# without wrapping the macro in a block, we'd need to choose different
# names for `a` and `b` here to avoid redefinition errors
for a, b in enumerate(10, [1, 2, 3, 5]):
  echo a, " ", b
```
