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



### Threads and exceptions

The interaction between threads and exceptions is simple: A *handled*
exception in one thread cannot affect any other thread. However, an
*unhandled* exception in one thread terminates the whole *process*.
