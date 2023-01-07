# `var`: mutable variables
This is for variables whose value can be modified after initialization:
<!---
```nim
var
  name, surname:string   # `name` and `surname` are defined as `string` type.
  age:int                # `age` will contain an `int`. 
  min:int = 50           # initilized to 50
  max     = 60           # Nim is capable of inferring the type

echo "Age: ", age        # by default initialized to 0

name = "Joe"             # initialization
surname = "Doe"
age = 15                 # age gets modified here (mutable)

echo name, " ", surname, ": ", age, " years old" 
```
-->
<iframe width=700, height=700 frameBorder=0 src="https://play.nim-lang.org/#ix=4krp"></iframe>


> Take a look at the [var statement](https://nim-lang.org/docs/manual.html#statements-and-expressions-var-statement) in the manual.

# `let`: immutable variables
Even better to use `let` when we don't expect that a variable will change. Modifications will be detected at compile time:
<!---
```nim
let  
  min = 10
  tolerance:float  # initialization is mandatory; comment this line to avoid the error
  tol = 0.1

min = 50   # mutation; comment this line to prevent the compile time error
echo min

```
-->

<iframe width=700, height=500 frameBorder=0 src="https://play.nim-lang.org/#ix=4kry"></iframe>


Nim supports three different types of variables, `let`, `var`, and `const`. As with most things, multiple variables can be declared in the same section.
<!--
```nim
proc getAlphabet(): string =
  var accm = ""
  for letter in 'a'..'z':  # see iterators
    accm.add(letter)
  return accm

# Computed at compilation time
const alphabet = getAlphabet()

# Compile-time error, const cannot be modified at run-time
alphabet = "abc"
```
-->
<!--
a.add("bar")
b += 1
-->



<!--
```
nim c --verbosity:2 ./variables.nim
variables.nim(22, 2) Error: 'let' symbol requires an initialization
    f: float
    ^
```

Without `--verbosity:2` only the error will be shown without the position cursor.
-->

# Const
A const variable’s value will be evaluated at compile-time, so if you inspect the C sources, you’ll see the following line:
```c
STRING_LITERAL(TMP129, "abcdefghijklmnopqrstuvwxyz", 26);
```
The only limitation with const is that compile-time evaluation cannot interface with C because there is no compile-time foreign function interface at this time.



<iframe width=700, height=1000 frameBorder=0 src="https://play.nim-lang.org/#ix=4khq"></iframe>
