
This is sort of `if`-`elif`-`else` chained constructs. All possible cases are required to be covered:









```nim
let value = "3"
case value:
  of "0":
    echo "Number: ", 0
  of "1", "2":
    echo "1-2"
  of "3":
    echo 3
  else:
    echo "value > 3"
```

<!--

case 'h':
  of 'a', 'e', 'i', 'o', 'u':
    echo "Vowel"
  of '\127'..'\255':
    echo "Unknown"
  else:
    echo "Consonant"

proc positiveOrNegative(num: int): string =
  result = case num:
    of low(int).. -1:
      "negative"
    of 0:
      "zero"
    of 1..high(int):
      "positive"
    else:
      "impossible"

echo positiveOrNegative(-1)
```

```sh
nim c -r ./case_stmts.nim
C
Consonant
negative
```
-->


<!--

- You can use strings in the switch statement
- Sets and ranges of ordinal types are also usable
- case statements, like most things, are actually expressions
-->