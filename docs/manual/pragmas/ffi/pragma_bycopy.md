### Bycopy pragma

The `bycopy` pragma can be applied to an object or tuple type and
instructs the compiler to pass the type by value to procs:

``` nim
type
Vector {.bycopy.} = object
x, y, z: float
```


