### linearScanEnd pragma

The `linearScanEnd` pragma can be used to tell the compiler how to
compile a Nim `case` statement.
Syntactically it has to be used as a statement:

```nim
case myInt
of 0:
echo "most common case"
of 1:
{.linearScanEnd.}
echo "second most common case"
of 2: echo "unlikely: use branch table"
else: echo "unlikely too: use branch table for ", myInt
```

In the example, the case branches `0` and `1` are much more common than
the other cases. Therefore the generated assembler code should test for
these values first so that the CPU\'s branch predictor has a good chance
to succeed (avoiding an expensive CPU pipeline stall). The other cases
might be put into a jump table for O(1) overhead but at the cost of a
(very likely) pipeline stall.

The `linearScanEnd` pragma should be put into the last branch that
should be tested against via linear scanning. If put into the last
branch of the whole `case` statement, the whole `case` statement uses
linear scanning.

