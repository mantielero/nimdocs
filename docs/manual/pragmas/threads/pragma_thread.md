### Thread pragma

A proc that is executed as a new thread of execution should be marked by
the `thread` pragma for reasons of readability. The compiler checks for
violations of the `no heap sharing restriction`{.interpreted-text
role="idx"}: This restriction implies that it is invalid to construct a
data structure that consists of memory allocated from different
(thread-local) heaps.

A thread proc is passed to `createThread` or `spawn` and invoked
indirectly; so the `thread` pragma implies `procvar`.