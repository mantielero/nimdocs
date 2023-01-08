Nim has several primitive types:


- floating points numbers: float32, float64, and float, where float is the processor’s fastest type
- characters: char, which is basically an alias for uint8

# Integers
There are signed and unsigned integers of different sizes:

- signed integers: `int8`, `int16`, `int32`, `int64`, and `int`, where `int` is the same size as a `pointer`
- unsigned integers: `uint8`, `uint16`, `uint32`, `uint64`, and `uint`

These types can be specified as follows (the shorthand adding the type after the number):
```nim
let
  a:int8   = -4
  b:uint16 =  20
  c        = -500000'i64
  d        =  573'u32
```
The type can be inferred most of the times.

???+ hint  "`/` vs `div`"
    Difference:

    - `/`: returns a floating point (even with integers operands)
    - `div`: returns an integer.   


## `hex`, `octal`, or `binary` literals
```nim
let
  a: int8  = 0x7F        # Hexadecimal
  b: uint8 = 0b1111_1111 # Binary; underscores can help with readability
  d        = 0xFF        # type is int
  c: uint8 = 256         # Compile time error
```


  

# Precedence 
Precedence rules are the same as in most other languages, but instead of `^`, `&`, `|`, `>>`, `<<`, the `xor`, `and`, `or`, `shr`, `shl` operators are used, respectively.
```nim
let
  a: int = 2
  b: int = 4
echo 4/2
```


```
nim c -r numbers2.nim
2.0
```

Another difference that may be surprising is that 