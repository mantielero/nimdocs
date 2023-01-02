## Overload disambiguation

For routine calls \"overload resolution\" is performed. There is a
weaker form of overload resolution called *overload disambiguation* that
is performed when an overloaded symbol is used in a context where there
is additional type information available. Let `p` be an overloaded
symbol. These contexts are:

-   In a function call `q(..., p, ...)` when the corresponding formal
    parameter of `q` is a `proc` type. If `q` itself is overloaded then
    the cartesian product of every interpretation of `q` and `p` must be
    considered.
-   In an object constructor `Obj(..., field: p, ...)` when `field` is a
    `proc` type. Analogous rules exist for array/set/tuple constructors.
-   In a declaration like `x: T = p` when `T` is a `proc` type.

As usual, ambiguous matches produce a compile-time error.