It creates a conditional loop:
```nim
var i = 0
while i < 10:
  echo i
  i += 1
```

You can create an infinite loop like:
```nim
while true:
  echo "."
```

# `break`

Immediately leaves the loop body.


```nim
var i = 0
while true:
  if i > 5:
    break

echo i
```

If you are using nested loops, you can use labeled blocks to indicate to which block it will break:
```nim
# This code will print "outer"
block outer:
  while true:
    block inner:
        while true:
            break inner
            #break outer
    echo "inner"
    break outer

echo "outer"
```
<!--
```nim
import strutils, random

randomize()
let answer = rand(10)
while true:
  echo "I have a number from 0 to 10, what is it? "
  let guess = parseInt(stdin.readLine)

  if guess < answer:
    echo "Too low, try again"
  elif guess > answer:
    echo "Too high, try again"
  else:
    echo "Correct!"
    break

block busyloops:
  while true:
    while true:
      break busyloops
```
-->

# `continue`
`continue` skips the rest of the loop body for a new iteration.

```nim
var i = 0
while true:
  if i > 3 and i <= 5:
    i += 1      # required to avoid infinite loop
    continue
  elif i > 5:
    break
  echo "inner ", i
  i += 1
  # `continue` will take you here
  
echo "end"
```