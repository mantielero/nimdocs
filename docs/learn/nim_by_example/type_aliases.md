# Type declarations
Types are declared inside type sections. Multiple types can be declared there. 


## Aliased types
Note that aliased types are the same, and not in any way incompatible with their original type. 
```nim
type
  Age* = int  # `Age` is an alias for `int`

let 
  a: int = 10
  b: Age = 20
echo a + b
```

## Distinct types
This provides type safety (not allowing to mix types):
```nim
type
  Age = distinct int
let 
  a: int = 10
  #b:Age  = 20 # ERROR
  b: Age = 20.Age
#echo a + b      # ERROR: type mismatch
#echo a.Age + b  # ERROR: type mismatch (sum operation does not exist for Age types)
echo a + b.int  # OK
```

???+ "hint" "`borrow` pragma
   This avoids having to create procedures for the operations related for the new types. For example:
   ```nim
   proc `+` *(a, b: Age): Age {.borrow.}
   proc `-` *(a, b: Age): Age {.borrow.}
   ```
   