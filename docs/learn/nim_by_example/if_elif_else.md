Nim uses `if`, `elif` and `else` for conditional execution of blocks of code:
```nim
let val = 10

if val < 5:
  echo "Smaller than 5"
elif val >= 5 and val < 10:
  echo "Equal or greater than 5 and smaller than 10"
else:
  echo "Equal or greater than 10"
```

???+ warning

    Note than Nim uses `elif` not `else if`.


Note that you can do:
```nim
let val = 10

let age = if val < 16:
            "kid"
          else:
            "adult"

echo age
```



