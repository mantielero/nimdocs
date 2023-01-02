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