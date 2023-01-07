Functions are named procedures in Nim.

A function with no input parameters and no return values would look like:
```nim
proc main =             # "main" procedure (no inputs, no outputs)
  # Here starts the function body
  echo "Hello world"    

main                   # Here we are calling "main"; "main()" would be valid too
```
> the function body is indented like in python


# Returning values
When the `proc` returns a value we need to annotate its type:
```nim
proc getValue():int =
  #             ^^^ return type 
  var a = 50
  a = a + 20
  a += 10
  return a

echo getValue()
```

The following is equivalent:
```nim
proc getValue():int =
  50 + 20 + 10  # The last statement is returned; no need for `return` keyword

echo getValue()
```

and the following:
```nim
proc getValue():int = 50 + 20 + 10

echo getValue()
```

# Input parameters
Input parameters need to be type annotated as well.

One input parameter and no returned value would be:
```nim
proc showValue(value:int) =
  #       param^^^^^ ^^^type
  echo "value: ", value

showValue(20)
```

Lets say we have two input parameters. We separate them using `;`:
```nim
proc showNameAndAge(name:string; age:int) =
  #           param1^^^^^^^^^^^  ^^^^^^^param2
  echo "Name: ", name, " is ", age, " years old"

showNameAndAge("Ana", 50)
```

When there are several parameters of the same type, we separate them using `,`:
```nim
proc showNameMinMax(name:string; min, max:int) =
  #                              ^^^^^^^^ two int params  
  echo "Name: ", name
  echo min, "<=", max

showNameMinMax("Ana", 10, 20)
```
# input and output parameters
An example mixing input and output parameters:
```nim
proc sum(a,b: int): string = 
  return "Total: " & $(a+b)

echo sum(10,20)
```

and another:
```nim
proc sum(a,b: int): int = a + b

echo sum(10,20)
```

# `discard` returned values
Whenever a procedure returns a value, it needs to be used. The following produces an error:
```nim
proc sum(a,b: int): int = a + b

sum(10,20)   # This will an error
```

You need to do something with the returned value:
```nim
proc sum(a,b: int): int = a + b

let value = sum(10,20) 
```

Or you can just do nothing:
```nim
proc sum(a,b: int): int = a + b

discard sum(10,20) 
```
> Discarding a value might be useful when you are just interested in the side effects.

# `result`variable
This is a special variable with the return type and whatever it holds will be returned.

```nim
proc example(val:int):int = 
  result = val + 20  #^^^this is the result's type
  echo "This is printed first"

echo example(10)
```

???+ warning "`result` shawdowing"

    If you define a variable named `result` within the function body it will shadow the special variable named `result`. So you won't get the expected behaviour:
    ```nim
    proc example(val:int):int = 
      let result:int = val + 20  
      
    echo example(10) # will print 0
    ```


<!--
They are declared using `proc` and require that their parameter and return types be annotated. After the types and parameters, an `=` is used to denote the start of the function body. Another thing to note is that procedures have uniform function call syntax, which means that they can called as both foo(a, b) or a.foo(b).
```nim
proc fibonacci(n: int): int =
  if n < 2:
    result = n
  else:
    result = fibonacci(n - 1) + (n - 2).fibonacci
```

# Exporting symbols
Encapsulation is also supported, not by conventions such as prepending the name with underscores but by annotating a procedure with *, which exports it and makes it available for use by modules.
```nim
# module1:
proc foo*(): int = 2
proc bar(): int = 3

# module2:
echo foo()  # Valid
echo bar()  # will not compile
```

# Side effect analyses
Nim provides support for functional programming and so includes the {.noSideEffect.} pragma, which statically ensures there are no side effects.
```nim
proc sum(x, y: int): int {. noSideEffect .} =
  x + y

proc minus(x, y: int): int {. noSideEffect .} =
  echo x  # error: 'minus' can have side effects
  x - y
```

# Operators
To create an operator, the symbols that are to be used must be encased inside `s to signify they are operators.
```nim
proc `$`(a: array[2, array[2, int]]): string =
  result = ""
  for v in a:
    for vx in v:
      result.add($vx & ", ")
    result.add("\n")

echo([[1, 2], [3, 4]])  # See varargs for
                        # how echo works

proc `^&*^@%`(a, b: string): string =
  ## A confusingly named useless operator
  result = a[0] & b[high(b)]

assert("foo" ^&*^@% "bar" == "fr")
```

# Generic Functions
Generic functions are like C++’s templates and allow for the same statically checked duck-typing semantics as templates.
```nim
proc `+`(a, b: string): string =
  a & b

proc `*`[T](a: T, b: int): T =
  result = default(T)
  for i in 0..b-1:
    result = result + a  # calls `+` from line 2

assert("a" * 10 == "aaaaaaaaaa")
```
-->