## Implementation Specific Pragmas

This section describes additional pragmas that the current Nim
implementation supports but which should not be seen as part of the
language specification.

### Bitsize pragma

The `bitsize` pragma is for object field members. It declares the field
as a bitfield in C/C++.

``` Nim
type
mybitfield = object
flag {.bitsize:1.}: cuint
```

generates:

``` C
struct mybitfield {
unsigned int flag:1;
};
```

### Align pragma

The `align` pragma is for variables and
object field members. It modifies the alignment requirement of the
entity being declared. The argument must be a constant power of 2. Valid
non-zero alignments that are weaker than other align pragmas on the same
declaration are ignored. Alignments that are weaker than the alignment
requirement of the type are ignored.

``` Nim
type
  sseType = object
    sseData {.align(16).}: array[4, float32]

  # every object will be aligned to 128-byte boundary
  Data = object
    x: char
    cacheline {.align(128).}: array[128, char] # over-aligned array of char,

proc main() =
  echo "sizeof(Data) = ", sizeof(Data), " (1 byte + 127 bytes padding + 128-byte array)"
  # output: sizeof(Data) = 256 (1 byte + 127 bytes padding + 128-byte array)
  echo "alignment of sseType is ", alignof(sseType)
  # output: alignment of sseType is 16
  var d {.align(2048).}: Data # this instance of data is aligned even stricter

main()
```

This pragma has no effect on the JS backend.

### Volatile pragma

The `volatile` pragma is for variables only. It declares the variable as
`volatile`{.interpreted-text role="c"}, whatever that means in C/C++
(its semantics are not well defined in C/C++).

**Note**: This pragma will not exist for the LLVM backend.

### nodecl pragma

The `nodecl` pragma can be applied to almost any symbol (variable, proc,
type, etc.) and is sometimes useful for interoperability with C: It
tells Nim that it should not generate a declaration for the symbol in
the C code. For example:

``` Nim
var
EACCES {.importc, nodecl.}: cint # pretend EACCES was a variable, as
# Nim does not know its value
```

However, the `header` pragma is often the better alternative.

**Note**: This will not work for the LLVM backend.

### Header pragma

The `header` pragma is very similar to the `nodecl` pragma: It can be
applied to almost any symbol and specifies that it should not be
declared and instead, the generated code should contain an
`#include`{.interpreted-text role="c"}:

``` Nim
type
PFile {.importc: "FILE*", header: "<stdio.h>".} = distinct pointer
# import C's FILE* type; Nim will treat it as a new pointer type
```

The `header` pragma always expects a string constant. The string
constant contains the header file: As usual for C, a system header file
is enclosed in angle brackets: `<>`{.interpreted-text role="c"}. If no
angle brackets are given, Nim encloses the header file in
`""`{.interpreted-text role="c"} in the generated C code.

**Note**: This will not work for the LLVM backend.

### IncompleteStruct pragma

The `incompleteStruct` pragma tells the compiler to not use the
underlying C `struct`{.interpreted-text role="c"} in a `sizeof`
expression:

``` Nim
type
DIR* {.importc: "DIR", header: "<dirent.h>",
pure, incompleteStruct.} = object
```

### Compile pragma

The `compile` pragma can be used to compile and link a C/C++ source file
with the project:

``` Nim
{.compile: "myfile.cpp".}
```

**Note**: Nim computes a SHA1 checksum and only recompiles the file if
it has changed. One can use the `-f`{.interpreted-text role="option"}
command-line option to force the recompilation of the file.

Since 1.4 the `compile` pragma is also available with this syntax:

``` Nim
{.compile("myfile.cpp", "--custom flags here").}
```

As can be seen in the example, this new variant allows for custom flags
that are passed to the C compiler when the file is recompiled.

### Link pragma

The `link` pragma can be used to link an additional file with the
project:

``` Nim
{.link: "myfile.o".}
```

### passc pragma

The `passc` pragma can be used to pass additional parameters to the C
compiler like one would using the command-line switch
`--passc`{.interpreted-text role="option"}:

``` Nim
{.passc: "-Wall -Werror".}
```

Note that one can use `gorge` from the [system module](system.html) to
embed parameters from an external command that will be executed during
semantic analysis:

``` Nim
{.passc: gorge("pkg-config --cflags sdl").}
```

### LocalPassc pragma

The `localPassc` pragma can be used to pass additional parameters to the
C compiler, but only for the C/C++ file that is produced from the Nim
module the pragma resides in:

``` Nim
# Module A.nim
# Produces: A.nim.cpp
{.localPassc: "-Wall -Werror".} # Passed when compiling A.nim.cpp
```

### passl pragma

The `passl` pragma can be used to pass additional parameters to the
linker like one would be using the command-line switch
`--passl`{.interpreted-text role="option"}:

``` Nim
{.passl: "-lSDLmain -lSDL".}
```

Note that one can use `gorge` from the [system module](system.html) to
embed parameters from an external command that will be executed during
semantic analysis:

``` Nim
{.passl: gorge("pkg-config --libs sdl").}
```

### Emit pragma

The `emit` pragma can be used to directly affect the output of the
compiler\'s code generator. The code is then unportable to other code
generators/backends. Its usage is highly discouraged! However, it can be
extremely useful for interfacing with `C++`{.interpreted-text
role="idx"} or `Objective C` code.

Example:

``` Nim
{.emit: """
static int cvariable = 420;
""".}
{.push stackTrace:off.}
proc embedsC() =
  var nimVar = 89
  # access Nim symbols within an emit section outside of string literals:
  {.emit: ["""fprintf(stdout, "%d\n", cvariable + (int)""", nimVar, ");"].}
{.pop.}

embedsC()
```

`nimbase.h` defines `NIM_EXTERNC`{.interpreted-text role="c"} C macro
that can be used for `extern "C"`{.interpreted-text role="cpp"} code to
work with both `nim c`{.interpreted-text role="cmd"} and
`nim cpp`{.interpreted-text role="cmd"}, e.g.:

``` Nim
proc foobar() {.importc:"$1".}
{.emit: """
#include <stdio.h>
NIM_EXTERNC
void fun(){}
""".}
```

::: note
::: title
Note
:::

For backward compatibility, if the argument to the `emit` statement is a
single string literal, Nim symbols can be referred to via backticks.
This usage is however deprecated.
:::

For a top-level emit statement, the section where in the generated C/C++
file the code should be emitted can be influenced via the prefixes
`/*TYPESECTION*/`{.interpreted-text role="c"} or
`/*VARSECTION*/`{.interpreted-text role="c"} or
`/*INCLUDESECTION*/`{.interpreted-text role="c"}:

``` Nim
{.emit: """/*TYPESECTION*/
struct Vector3 {
public:
Vector3(): x(5) {}
Vector3(float x_): x(x_) {}
float x;
};
""".}
type Vector3 {.importcpp: "Vector3", nodecl} = object
  x: cfloat

proc constructVector3(a: cfloat): Vector3 {.importcpp: "Vector3(@)", nodecl}
```

### ImportCpp pragma

**Note**:
[c2nim](https://github.com/nim-lang/c2nim/blob/master/doc/c2nim.rst) can
parse a large subset of C++ and knows about the `importcpp` pragma
pattern language. It is not necessary to know all the details described
here.

Similar to the [importc pragma for
C](#foreign-function-interface-importc-pragma), the `importcpp` pragma
can be used to import `C++` methods or C++
symbols in general. The generated code then uses the C++ method calling
syntax: `obj->method(arg)`{.interpreted-text role="cpp"}. In combination
with the `header` and `emit` pragmas this allows *sloppy* interfacing
with libraries written in C++:

``` Nim
# Horrible example of how to interface with a C++ engine ... ;-)
{.link: "/usr/lib/libIrrlicht.so".}

{.emit: """
using namespace irr;
using namespace core;
using namespace scene;
using namespace video;
using namespace io;
using namespace gui;
""".}

const
  irr = "<irrlicht/irrlicht.h>"

type
  IrrlichtDeviceObj {.header: irr,
                      importcpp: "IrrlichtDevice".} = object
  IrrlichtDevice = ptr IrrlichtDeviceObj

proc createDevice(): IrrlichtDevice {.
  header: irr, importcpp: "createDevice(@)".}
proc run(device: IrrlichtDevice): bool {.
  header: irr, importcpp: "#.run(@)".}
```

The compiler needs to be told to generate C++ (command
`cpp`{.interpreted-text role="option"}) for this to work. The
conditional symbol `cpp` is defined when the compiler emits C++ code.

#### Namespaces

The *sloppy interfacing* example uses `.emit` to produce
`using namespace`{.interpreted-text role="cpp"} declarations. It is
usually much better to instead refer to the imported name via the
`namespace::identifier`{.interpreted-text role="cpp"} notation:

``` nim
type
IrrlichtDeviceObj {.header: irr,
importcpp: "irr::IrrlichtDevice".} = object
```

#### Importcpp for enums

When `importcpp` is applied to an enum type the numerical enum values
are annotated with the C++ enum type, like in this example:
`((TheCppEnum)(3))`{.interpreted-text role="cpp"}. (This turned out to
be the simplest way to implement it.)

#### Importcpp for procs

Note that the `importcpp` variant for procs uses a somewhat cryptic
pattern language for maximum flexibility:

-   A hash `#` symbol is replaced by the first or next argument.
-   A dot following the hash `#.` indicates that the call should use
    C++\'s dot or arrow notation.
-   An at symbol `@` is replaced by the remaining arguments, separated
    by commas.

For example:

``` nim
proc cppMethod(this: CppObj, a, b, c: cint) {.importcpp: "#.CppMethod(@)".}
var x: ptr CppObj
cppMethod(x[], 1, 2, 3)
```

Produces:

``` C
x->CppMethod(1, 2, 3)
```

As a special rule to keep backward compatibility with older versions of
the `importcpp` pragma, if there is no special pattern character (any of
`# ' @`) at all, C++\'s dot or arrow notation is assumed, so the above
example can also be written as:

``` nim
proc cppMethod(this: CppObj, a, b, c: cint) {.importcpp: "CppMethod".}
```

Note that the pattern language naturally also covers C++\'s operator
overloading capabilities:

``` nim
proc vectorAddition(a, b: Vec3): Vec3 {.importcpp: "# + #".}
proc dictLookup(a: Dict, k: Key): Value {.importcpp: "#[#]".}
```

-   An apostrophe `'` followed by an integer `i` in the range 0..9 is
    replaced by the i\'th parameter *type*. The 0th position is the
    result type. This can be used to pass types to C++ function
    templates. Between the `'` and the digit, an asterisk can be used to
    get to the base type of the type. (So it \"takes away a star\" from
    the type; `T*`{.interpreted-text role="c"} becomes `T`.) Two stars
    can be used to get to the element type of the element type etc.

For example:

``` nim
type Input {.importcpp: "System::Input".} = object
proc getSubsystem*[T](): ptr T {.importcpp: "SystemManager::getSubsystem<'*0>()", nodecl.}

let x: ptr Input = getSubsystem[Input]()
```

Produces:

``` C
x = SystemManager::getSubsystem<System::Input>()
```

-   `#@` is a special case to support a `cnew` operation. It is required
    so that the call expression is inlined directly, without going
    through a temporary location. This is only required to circumvent a
    limitation of the current code generator.

For example C++\'s `new`{.interpreted-text role="cpp"} operator can be
\"imported\" like this:

``` nim
proc cnew*[T](x: T): ptr T {.importcpp: "(new '*0#@)", nodecl.}
# constructor of 'Foo':
proc constructFoo(a, b: cint): Foo {.importcpp: "Foo(@)".}

let x = cnew constructFoo(3, 4)
```

Produces:

``` C
x = new Foo(3, 4)
```

However, depending on the use case `new Foo`{.interpreted-text
role="cpp"} can also be wrapped like this instead:

``` nim
proc newFoo(a, b: cint): ptr Foo {.importcpp: "new Foo(@)".}
let x = newFoo(3, 4)
```

#### Wrapping constructors

Sometimes a C++ class has a private copy constructor and so code like
`Class c = Class(1,2);`{.interpreted-text role="cpp"} must not be
generated but instead `Class c(1,2);`{.interpreted-text role="cpp"}. For
this purpose the Nim proc that wraps a C++ constructor needs to be
annotated with the `constructor` pragma.
This pragma also helps to generate faster C++ code since construction
then doesn\'t invoke the copy constructor:

``` nim
# a better constructor of 'Foo':
proc constructFoo(a, b: cint): Foo {.importcpp: "Foo(@)", constructor.}
```

#### Wrapping destructors

Since Nim generates C++ directly, any destructor is called implicitly by
the C++ compiler at the scope exits. This means that often one can get
away with not wrapping the destructor at all! However, when it needs to
be invoked explicitly, it needs to be wrapped. The pattern language
provides everything that is required:

``` nim
proc destroyFoo(this: var Foo) {.importcpp: "#.~Foo()".}
```

#### Importcpp for objects

Generic `importcpp`\'ed objects are mapped to C++ templates. This means
that one can import C++\'s templates rather easily without the need for
a pattern language for object types:

``` {.nim test="\"nim cpp $1\""}
type
  StdMap[K, V] {.importcpp: "std::map", header: "<map>".} = object
proc `[]=`[K, V](this: var StdMap[K, V]; key: K; val: V) {.
  importcpp: "#[#] = #", header: "<map>".}

var x: StdMap[cint, cdouble]
x[6] = 91.4
```

Produces:

``` C
std::map<int, double> x;
x[6] = 91.4;
```

-   If more precise control is needed, the apostrophe `'` can be used in
    the supplied pattern to denote the concrete type parameters of the
    generic type. See the usage of the apostrophe operator in proc
    patterns for more details.

    ``` nim
    type
      VectorIterator {.importcpp: "std::vector<'0>::iterator".} [T] = object

    var x: VectorIterator[cint]
    ```

    Produces:

    ``` C
    std::vector<int>::iterator x;
    ```

### ImportJs pragma

Similar to the [importcpp pragma for
C++](#implementation-specific-pragmas-importcpp-pragma), the `importjs`
pragma can be used to import Javascript methods or symbols in general.
The generated code then uses the Javascript method calling syntax:
`obj.method(arg)`.

### ImportObjC pragma

Similar to the [importc pragma for
C](#foreign-function-interface-importc-pragma), the `importobjc` pragma
can be used to import `Objective C`
methods. The generated code then uses the Objective C method calling
syntax: `[obj method param1: arg]`. In addition with the `header` and
`emit` pragmas this allows *sloppy* interfacing with libraries written
in Objective C:

``` Nim
# horrible example of how to interface with GNUStep ...
{.passl: "-lobjc".}
{.emit: """
#include <objc/Object.h>
@interface Greeter:Object
{
}

- (void)greet:(long)x y:(long)dummy;
@end

#include <stdio.h>
@implementation Greeter

- (void)greet:(long)x y:(long)dummy
{
  printf("Hello, World!\n");
}
@end

#include <stdlib.h>
""".}

type
  Id {.importc: "id", header: "<objc/Object.h>", final.} = distinct int

proc newGreeter: Id {.importobjc: "Greeter new", nodecl.}
proc greet(self: Id, x, y: int) {.importobjc: "greet", nodecl.}
proc free(self: Id) {.importobjc: "free", nodecl.}

var g = newGreeter()
g.greet(12, 34)
g.free()
```

The compiler needs to be told to generate Objective C (command
`objc`{.interpreted-text role="option"}) for this to work. The
conditional symbol `objc` is defined when the compiler emits Objective C
code.

### CodegenDecl pragma

The `codegenDecl` pragma can be used to directly influence Nim\'s code
generator. It receives a format string that determines how the variable
or proc is declared in the generated code.

For variables, \$1 in the format string represents the type of the
variable and \$2 is the name of the variable.

The following Nim code:

``` nim
var
a {.codegenDecl: "$# progmem $#".}: int
```

will generate this C code:

``` c
int progmem a
```

For procedures, \$1 is the return type of the procedure, \$2 is the name
of the procedure, and \$3 is the parameter list.

The following nim code:

``` nim
proc myinterrupt() {.codegenDecl: "__interrupt $# $#$#".} =
echo "realistic interrupt handler"
```

will generate this code:

``` c
__interrupt void myinterrupt()
```

### `cppNonPod` pragma

The `.cppNonPod` pragma should be used for non-POD `importcpp` types so
that they work properly (in particular regarding constructor and
destructor) for `.threadvar` variables. This requires
`--tlsEmulation:off`{.interpreted-text role="option"}.

``` nim
type Foo {.cppNonPod, importcpp, header: "funs.h".} = object
x: cint
proc main()=
var a {.threadvar.}: Foo
```

### compile-time define pragmas

The pragmas listed here can be used to optionally accept values from the
`-d/--define`{.interpreted-text role="option"} option at compile time.

The implementation currently provides the following possible options
(various others may be added later).

  pragma                                       description
  -------------------------------------------- --------------------------------------------
  `intdefine`    Reads in a build-time define as an integer
  `strdefine`    Reads in a build-time define as a string
  `booldefine`   Reads in a build-time define as a bool

``` nim
const FooBar {.intdefine.}: int = 5
echo FooBar
```

``` cmd
nim c -d:FooBar=42 foobar.nim
```

In the above example, providing the `-d`{.interpreted-text
role="option"} flag causes the symbol `FooBar` to be overwritten at
compile-time, printing out 42. If the `-d:FooBar=42`{.interpreted-text
role="option"} were to be omitted, the default value of 5 would be used.
To see if a value was provided, `defined(FooBar)` can be used.

The syntax `-d:flag`{.interpreted-text role="option"} is actually just a
shortcut for `-d:flag=true`{.interpreted-text role="option"}.

