### experimental pragma

The `experimental` pragma enables experimental language features.
Depending on the concrete feature, this means that the feature is either
considered too unstable for an otherwise stable release or that the
future of the feature is uncertain (it may be removed at any time).

Example:

```nim
import std/threadpool
{.experimental: "parallel".}
proc threadedEcho(s: string, i: int) =
  echo(s, " ", $i)

proc useParallel() =
  parallel:
    for i in 0..4:
      spawn threadedEcho("echo in parallel", i)

useParallel()
```

As a top-level statement, the experimental pragma enables a feature for
the rest of the module it\'s enabled in. This is problematic for macro
and generic instantiations that cross a module scope. Currently, these
usages have to be put into a `.push/pop` environment:

```nim
# client.nim
proc useParallel*[T](unused: T) =
  # use a generic T here to show the problem.
  {.push experimental: "parallel".}
  parallel:
    for i in 0..4:
      echo "echo in parallel"

  {.pop.}
```

```nim
import client
useParallel(1)
```