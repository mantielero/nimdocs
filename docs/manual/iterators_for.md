## Iterators and the for statement

The `for` statement is an abstract
mechanism to iterate over the elements of a container. It relies on an
`iterator` to do so. Like `while`
statements, `for` statements open an `implicit block`{.interpreted-text
role="idx"} so that they can be left with a `break` statement.

The `for` loop declares iteration variables - their scope reaches until
the end of the loop body. The iteration variables\' types are inferred
by the return type of the iterator.

An iterator is similar to a procedure, except that it can be called in
the context of a `for` loop. Iterators provide a way to specify the
iteration over an abstract type. The `yield` statement in the called
iterator plays a key role in the execution of a `for` loop. Whenever a
`yield` statement is reached, the data is bound to the `for` loop
variables and control continues in the body of the `for` loop. The
iterator\'s local variables and execution state are automatically saved
between calls. Example:

``` nim
# this definition exists in the system module
iterator items*(a: string): char {.inline.} =
var i = 0
while i < len(a):
yield a[i]
inc(i)
for ch in items("hello world"): # `ch` is an iteration variable
  echo ch
```

The compiler generates code as if the programmer would have written
this:

``` nim
var i = 0
while i < len(a):
var ch = a[i]
echo ch
inc(i)
```

If the iterator yields a tuple, there can be as many iteration variables
as there are components in the tuple. The i\'th iteration variable\'s
type is the type of the i\'th component. In other words, implicit tuple
unpacking in a for loop context is supported.

### Implicit items/pairs invocations

If the for loop expression `e` does not denote an iterator and the for
loop has exactly 1 variable, the for loop expression is rewritten to
`items(e)`; ie. an `items` iterator is implicitly invoked:

``` nim
for x in [1,2,3]: echo x
```

If the for loop has exactly 2 variables, a `pairs` iterator is
implicitly invoked.

Symbol lookup of the identifiers `items`/`pairs` is performed after the
rewriting step, so that all overloads of `items`/`pairs` are taken into
account.

### First-class iterators

There are 2 kinds of iterators in Nim: *inline* and *closure* iterators.
An `inline iterator` is an iterator
that\'s always inlined by the compiler leading to zero overhead for the
abstraction, but may result in a heavy increase in code size.

Caution: the body of a for loop over an inline iterator is inlined into
each `yield` statement appearing in the iterator code, so ideally the
code should be refactored to contain a single yield when possible to
avoid code bloat.

Inline iterators are second class citizens; They can be passed as
parameters only to other inlining code facilities like templates,
macros, and other inline iterators.

In contrast to that, a `closure iterator`
can be passed around more freely:

``` nim
iterator count0(): int {.closure.} =
yield 0
iterator count2(): int {.closure.} =
  var x = 1
  yield x
  inc x
  yield x

proc invoke(iter: iterator(): int {.closure.}) =
  for x in iter(): echo x

invoke(count0)
invoke(count2)
```

Closure iterators and inline iterators have some restrictions:

1.  For now, a closure iterator cannot be executed at compile time.
2.  `return` is allowed in a closure iterator but not in an inline
    iterator (but rarely useful) and ends the iteration.
3.  Neither inline nor closure iterators can be (directly)\* recursive.
4.  Neither inline nor closure iterators have the special `result`
    variable.
5.  Closure iterators are not supported by the JS backend.

(\*) Closure iterators can be co-recursive with a factory proc which
results in similar syntax to a recursive iterator. More details follow.

Iterators that are neither marked `{.closure.}` nor `{.inline.}`
explicitly default to being inline, but this may change in future
versions of the implementation.

The `iterator` type is always of the calling convention `closure`
implicitly; the following example shows how to use iterators to
implement a `collaborative tasking`
system:

``` nim
# simple tasking:
type
Task = iterator (ticker: int)
iterator a1(ticker: int) {.closure.} =
  echo "a1: A"
  yield
  echo "a1: B"
  yield
  echo "a1: C"
  yield
  echo "a1: D"

iterator a2(ticker: int) {.closure.} =
  echo "a2: A"
  yield
  echo "a2: B"
  yield
  echo "a2: C"

proc runTasks(t: varargs[Task]) =
  var ticker = 0
  while true:
    let x = t[ticker mod t.len]
    if finished(x): break
    x(ticker)
    inc ticker

runTasks(a1, a2)
```

The builtin `system.finished` can be used to determine if an iterator
has finished its operation; no exception is raised on an attempt to
invoke an iterator that has already finished its work.

Note that `system.finished` is error prone to use because it only
returns `true` one iteration after the iterator has finished:

``` nim
iterator mycount(a, b: int): int {.closure.} =
var x = a
while x <= b:
yield x
inc x
var c = mycount # instantiate the iterator
while not finished(c):
  echo c(1, 3)

# Produces
1
2
3
0
```

Instead this code has to be used:

``` nim
var c = mycount # instantiate the iterator
while true:
let value = c(1, 3)
if finished(c): break # and discard 'value'!
echo value
```

It helps to think that the iterator actually returns a pair
`(value, done)` and `finished` is used to access the hidden `done`
field.

Closure iterators are *resumable functions* and so one has to provide
the arguments to every call. To get around this limitation one can
capture parameters of an outer factory proc:

``` nim
proc mycount(a, b: int): iterator (): int =
result = iterator (): int =
var x = a
while x <= b:
yield x
inc x
let foo = mycount(1, 4)

for f in foo():
  echo f
```

The call can be made more like an inline iterator with a for loop macro:

``` nim
import std/macros
macro toItr(x: ForLoopStmt): untyped =
let expr = x[0]
let call = x[1][1] # Get foo out of toItr(foo)
let body = x[2]
result = quote do:
block:
let itr = `call`
for `expr` in itr():
`body`
for f in toItr(mycount(1, 4)): # using early `proc mycount`
  echo f
```

Because of full backend function call aparatus involvment, closure
iterator invocation is typically higher cost than inline iterators.
Adornment by a macro wrapper at the call site like this is a possibly
useful reminder.

The factory `proc`, as an ordinary procedure, can be recursive. The
above macro allows such recursion to look much like a recursive iterator
would. For example:

``` nim
proc recCountDown(n: int): iterator(): int =
result = iterator(): int =
if n > 0:
yield n
for e in toItr(recCountDown(n - 1)):
yield e
for i in toItr(recCountDown(6)): # Emits: 6 5 4 3 2 1
  echo i
```

See also see [iterable](#overloading-resolution-iterable) for passing
iterators to templates and macros.

