### compilation option pragmas

The listed pragmas here can be used to override the code generation
options for a proc/method/converter.

The implementation currently provides the following possible options
(various others may be added later).

  pragma           allowed values   description
  ---------------- ---------------- ------------------------------------------------------------------------------------------------
  checks           on\|off          Turns the code generation for all runtime checks on or off.
  boundChecks      on\|off          Turns the code generation for array bound checks on or off.
  overflowChecks   on\|off          Turns the code generation for over- or underflow checks on or off.
  nilChecks        on\|off          Turns the code generation for nil pointer checks on or off.
  assertions       on\|off          Turns the code generation for assertions on or off.
  warnings         on\|off          Turns the warning messages of the compiler on or off.
  hints            on\|off          Turns the hint messages of the compiler on or off.
  optimization     nonesize         Optimize the code for speed or size, or disable optimization.
  patterns         on\|off          Turns the term rewriting templates/macros on or off.
  callconv         cdecl\|\...      Specifies the default calling convention for all procedures (and procedure types) that follow.

Example:

```nim
{.checks: off, optimization: speed.}
# compile without runtime checks and optimize for speed
```

