## Constants and Constant Expressions

A `constant` is a symbol that is bound to
the value of a constant expression. Constant expressions are restricted
to depend only on the following categories of values and operations,
because these are either built into the language or declared and
evaluated before semantic analysis of the constant expression:

-   literals
-   built-in operators
-   previously declared constants and compile-time variables
-   previously declared macros and templates
-   previously declared procedures that have no side effects beyond
    possibly modifying compile-time variables

A constant expression can contain code blocks that may internally use
all Nim features supported at compile time (as detailed in the next
section below). Within such a code block, it is possible to declare
variables and then later read and update them, or declare variables and
pass them to procedures that modify them. However, the code in such a
block must still adhere to the restrictions listed above for referencing
values and operations outside the block.

The ability to access and modify compile-time variables adds flexibility
to constant expressions that may be surprising to those coming from
other statically typed languages. For example, the following code echoes
the beginning of the Fibonacci series **at compile-time**. (This is a
demonstration of flexibility in defining constants, not a recommended
style for solving this problem.)

```nim
import std/strformat

var fibN {.compileTime.}: int
var fibPrev {.compileTime.}: int
var fibPrevPrev {.compileTime.}: int

proc nextFib(): int =
  result = if fibN < 2:
    fibN
  else:
    fibPrevPrev + fibPrev
  inc(fibN)
  fibPrevPrev = fibPrev
  fibPrev = result

const f0 = nextFib()
const f1 = nextFib()

const displayFib = block:
  const f2 = nextFib()
  var result = fmt"Fibonacci sequence: {f0}, {f1}, {f2}"
  for i in 3..12:
    add(result, fmt", {nextFib()}")
  result

static:
  echo displayFib
```

