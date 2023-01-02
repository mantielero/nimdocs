### asmNoStackFrame pragma

A proc can be marked with the `asmNoStackFrame` pragma to tell the
compiler it should not generate a stack frame for the proc. There are
also no exit statements like `return result;` generated and the
generated C function is declared as
`__declspec(naked)`{.interpreted-text role="c"} or
`__attribute__((naked))`{.interpreted-text role="c"} (depending on the
used C compiler).

**Note**: This pragma should only be used by procs which consist solely
of assembler statements.

