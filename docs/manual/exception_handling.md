## Exception handling

### Try statement

Example:

``` nim
# read the first two lines of a text file that should contain numbers
# and tries to add them
var
f: File
if open(f, "numbers.txt"):
try:
var a = readLine(f)
var b = readLine(f)
echo "sum: " & $(parseInt(a) + parseInt(b))
except OverflowDefect:
echo "overflow!"
except ValueError, IOError:
echo "catch multiple exceptions!"
except CatchableError:
echo "Catchable exception!"
finally:
close(f)
```

The statements after the `try` are executed in sequential order unless
an exception `e` is raised. If the exception type of `e` matches any
listed in an `except` clause, the corresponding statements are executed.
The statements following the `except` clauses are called
`exception handlers`{.interpreted-text role="idx"}.

If there is a `finally`{.interpreted-text role="idx"} clause, it is
always executed after the exception handlers.

The exception is *consumed* in an exception handler. However, an
exception handler may raise another exception. If the exception is not
handled, it is propagated through the call stack. This means that often
the rest of the procedure - that is not within a `finally` clause -is
not executed (if an exception occurs).

### Try expression

Try can also be used as an expression; the type of the `try` branch then
needs to fit the types of `except` branches, but the type of the
`finally` branch always has to be \`void\`:

``` nim
from std/strutils import parseInt
let x = try: parseInt("133a")
        except ValueError: -1
        finally: echo "hi"
```

To prevent confusing code there is a parsing limitation; if the `try`
follows a `(` it has to be written as a one liner:

``` nim
from std/strutils import parseInt
let x = (try: parseInt("133a") except ValueError: -1)
```

### Except clauses

Within an `except` clause it is possible to access the current exception
using the following syntax:

``` nim
try:
# ...
except IOError as e:
# Now use "e"
echo "I/O error: " & e.msg
```

Alternatively, it is possible to use `getCurrentException` to retrieve
the exception that has been raised:

``` nim
try:
# ...
except IOError:
let e = getCurrentException()
# Now use "e"
```

Note that `getCurrentException` always returns a `ref Exception` type.
If a variable of the proper type is needed (in the example above,
`IOError`), one must convert it explicitly:

``` nim
try:
# ...
except IOError:
let e = (ref IOError)(getCurrentException())
# "e" is now of the proper type
```

However, this is seldom needed. The most common case is to extract an
error message from `e`, and for such situations, it is enough to use
\`getCurrentExceptionMsg\`:

``` nim
try:
# ...
except CatchableError:
echo getCurrentExceptionMsg()
```

### Custom exceptions

It is possible to create custom exceptions. A custom exception is a
custom type:

``` nim
type
LoadError* = object of Exception
```

Ending the custom exception\'s name with `Error` is recommended.

Custom exceptions can be raised just like any other exception, e.g.:

``` nim
raise newException(LoadError, "Failed to load data")
```

### Defer statement

Instead of a `try finally` statement a `defer` statement can be used,
which avoids lexical nesting and offers more flexibility in terms of
scoping as shown below.

Any statements following the `defer` in the current block will be
considered to be in an implicit try block:

``` {.nim test="\"nim c $1\""}
proc main =
  var f = open("numbers.txt", fmWrite)
  defer: close(f)
  f.write "abc"
  f.write "def"
```

Is rewritten to:

``` {.nim test="\"nim c $1\""}
proc main =
  var f = open("numbers.txt")
  try:
    f.write "abc"
    f.write "def"
  finally:
    close(f)
```

When `defer` is at the outermost scope of a template/macro, its scope
extends to the block where the template is called from:

``` {.nim test="\"nim c $1\""}
template safeOpenDefer(f, path) =
  var f = open(path, fmWrite)
  defer: close(f)

template safeOpenFinally(f, path, body) =
  var f = open(path, fmWrite)
  try: body # without `defer`, `body` must be specified as parameter
  finally: close(f)

block:
  safeOpenDefer(f, "/tmp/z01.txt")
  f.write "abc"
block:
  safeOpenFinally(f, "/tmp/z01.txt"):
    f.write "abc" # adds a lexical scope
block:
  var f = open("/tmp/z01.txt", fmWrite)
  try:
    f.write "abc" # adds a lexical scope
  finally: close(f)
```

Top-level `defer` statements are not supported since it\'s unclear what
such a statement should refer to.

### Raise statement

Example:

``` nim
raise newException(IOError, "IO failed")
```

Apart from built-in operations like array indexing, memory allocation,
etc. the `raise` statement is the only way to raise an exception.

If no exception name is given, the current exception is
`re-raised`{.interpreted-text role="idx"}. The
`ReraiseDefect`{.interpreted-text role="idx"} exception is raised if
there is no exception to re-raise. It follows that the `raise` statement
*always* raises an exception.

### Exception hierarchy

The exception tree is defined in the [system](system.html) module. Every
exception inherits from `system.Exception`. Exceptions that indicate
programming bugs inherit from `system.Defect` (which is a subtype of
`Exception`) and are strictly speaking not catchable as they can also be
mapped to an operation that terminates the whole process. If panics are
turned into exceptions, these exceptions inherit from `Defect`.

Exceptions that indicate any other runtime error that can be caught
inherit from `system.CatchableError` (which is a subtype of
`Exception`).

### Imported exceptions

It is possible to raise/catch imported C++ exceptions. Types imported
using `importcpp` can be raised or caught. Exceptions are raised by
value and caught by reference. Example:

``` {.nim test="\"nim cpp -r $1\""}
type
  CStdException {.importcpp: "std::exception", header: "<exception>", inheritable.} = object
    ## does not inherit from `RootObj`, so we use `inheritable` instead
  CRuntimeError {.requiresInit, importcpp: "std::runtime_error", header: "<stdexcept>".} = object of CStdException
    ## `CRuntimeError` has no default constructor => `requiresInit`
proc what(s: CStdException): cstring {.importcpp: "((char *)#.what())".}
proc initRuntimeError(a: cstring): CRuntimeError {.importcpp: "std::runtime_error(@)", constructor.}
proc initStdException(): CStdException {.importcpp: "std::exception()", constructor.}

proc fn() =
  let a = initRuntimeError("foo")
  doAssert $a.what == "foo"
  var b: cstring
  try: raise initRuntimeError("foo2")
  except CStdException as e:
    doAssert e is CStdException
    b = e.what()
  doAssert $b == "foo2"

  try: raise initStdException()
  except CStdException: discard

  try: raise initRuntimeError("foo3")
  except CRuntimeError as e:
    b = e.what()
  except CStdException:
    doAssert false
  doAssert $b == "foo3"

fn()
```

**Note:** `getCurrentException()` and `getCurrentExceptionMsg()` are not
available for imported exceptions from C++. One needs to use the
`except ImportedException as x:` syntax and rely on functionality of the
`x` object to get exception details.
