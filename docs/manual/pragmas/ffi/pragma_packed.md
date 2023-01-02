### Packed pragma

The `packed` pragma can be applied to any `object` type. It ensures that
the fields of an object are packed back-to-back in memory. It is useful
to store packets or messages from/to network or hardware drivers, and
for interoperability with C. Combining packed pragma with inheritance is
not defined, and it should not be used with GC\'ed memory (ref\'s).

**Future directions**: Using GC\'ed memory in packed pragma will result
in a static error. Usage with inheritance should be defined and
documented.

