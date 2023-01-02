## Threads

To enable thread support the `--threads:on`{.interpreted-text
role="option"} command-line switch needs to be used. The [system]()
module then contains several threading primitives. See the
[channels](channels_builtin.html) modules for the low-level thread API.
There are also high-level parallelism constructs available. See
[spawn](manual_experimental.html#parallel-amp-spawn) for further
details.

Nim\'s memory model for threads is quite different than that of other
common programming languages (C, Pascal, Java): Each thread has its own
(garbage collected) heap, and sharing of memory is restricted to global
variables. This helps to prevent race conditions. GC efficiency is
improved quite a lot, because the GC never has to stop other threads and
see what they reference.

The only way to create a thread is via `spawn` or `createThread`. The
invoked proc must not use `var` parameters nor must any of its
parameters contain a `ref` or `closure` type. This enforces the *no heap
sharing restriction*.

### Thread pragma

A proc that is executed as a new thread of execution should be marked by
the `thread` pragma for reasons of readability. The compiler checks for
violations of the `no heap sharing restriction`{.interpreted-text
role="idx"}: This restriction implies that it is invalid to construct a
data structure that consists of memory allocated from different
(thread-local) heaps.

A thread proc is passed to `createThread` or `spawn` and invoked
indirectly; so the `thread` pragma implies `procvar`.

### Threadvar pragma

A variable can be marked with the `threadvar` pragma, which makes it a
`thread-local` variable; Additionally,
this implies all the effects of the `global` pragma.

``` nim
var checkpoints* {.threadvar.}: seq[string]
```

Due to implementation restrictions, thread-local variables cannot be
initialized within the `var` section. (Every thread-local variable needs
to be replicated at thread creation.)

### Threads and exceptions

The interaction between threads and exceptions is simple: A *handled*
exception in one thread cannot affect any other thread. However, an
*unhandled* exception in one thread terminates the whole *process*.
