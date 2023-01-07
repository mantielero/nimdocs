# Indenting
We saw that the procedures body, was indented like in python. So, indenting is one way of creating code blocks. We can see a block of code highlighted in yellow in the next example:
```nim hl_lines="2 3 4"
proc example(val:int) = 
  # prints val plus 20
  let value = val + 20  
  echo value

example(10)
```

# `block` statement 
We can use `block` statement to create a code block:
```nim
block:
  echo "this is"
  echo "another block"
```

They can be labeled:
```nim
block yourname:
  echo "this is"
  echo "a block"
```

> we will come back to labeled blocks when we talk about loops





<!--
The block statement can also be labeled, making it useful for breaking out of loops and is useful for general scoping as well.
```nim
block outer:
  for i in 0..2000:
    for j in 0..2000:
      if i+j == 3145:
        echo i, ", ", j
        break outer

let b = 3
block:
  let b = "3"  # shadowing is probably a dumb idea
```
-->