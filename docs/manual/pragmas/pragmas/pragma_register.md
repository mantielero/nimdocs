### register pragma

The `register` pragma is for variables only. It declares the variable as
`register`, giving the compiler a hint that the variable should be
placed in a hardware register for faster access. C compilers usually
ignore this though and for good reasons: Often they do a better job
without it anyway.

However, in highly specific cases (a dispatch loop of a bytecode
interpreter for example) it may provide benefits.
