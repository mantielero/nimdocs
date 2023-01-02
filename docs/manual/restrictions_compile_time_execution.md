## Restrictions on Compile-Time Execution

Nim code that will be executed at compile time cannot use the following
language features:

-   methods
-   closure iterators
-   the `cast` operator
-   reference (pointer) types
-   FFI

The use of wrappers that use FFI and/or `cast` is also disallowed. Note
that these wrappers include the ones in the standard libraries.

Some or all of these restrictions are likely to be lifted over time.

