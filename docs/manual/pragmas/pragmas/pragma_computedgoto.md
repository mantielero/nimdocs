### computedGoto pragma

The `computedGoto` pragma can be used to tell the compiler how to
compile a Nim `case` in a `while true`
statement. Syntactically it has to be used as a statement inside the
loop:

```nim
type
  MyEnum = enum
    enumA, enumB, enumC, enumD, enumE

proc vm() =
  var instructions: array[0..100, MyEnum]
  instructions[2] = enumC
  instructions[3] = enumD
  instructions[4] = enumA
  instructions[5] = enumD
  instructions[6] = enumC
  instructions[7] = enumA
  instructions[8] = enumB

  instructions[12] = enumE
  var pc = 0
  while true:
    {.computedGoto.}
    let instr = instructions[pc]
    case instr
    of enumA:
      echo "yeah A"
    of enumC, enumD:
      echo "yeah CD"
    of enumB:
      echo "yeah B"
    of enumE:
      break
    inc(pc)

vm()
```

As the example shows, `computedGoto` is mostly useful for interpreters.
If the underlying backend (C compiler) does not support the computed
goto extension the pragma is simply ignored.

