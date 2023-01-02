### Union pragma

The `union` pragma can be applied to any `object` type. It means all of
the object\'s fields are overlaid in memory. This produces a
`union`{.interpreted-text role="c"} instead of a
`struct`{.interpreted-text role="c"} in the generated C/C++ code. The
object declaration then must not use inheritance or any GC\'ed memory
but this is currently not checked.

**Future directions**: GC\'ed memory should be allowed in unions and the
GC should scan unions conservatively.

