
# Types - smart pointers
The first line is quite heavyweight for a noob like me:
```c++
osg::ref_ptr<osg::Geometry> myTriangleGeometry = new osg::Geometry;
```
It is defining the variable `myTriangleGeometry`. The type of this variable is `osg::ref_ptr<osg::Geometry>`.... wow, what is this!! It seems that this pattern is very common in C++. Many libraries define their own *smart pointers*; this as pointers that handle memory automatically (allocates memory, and deallocates it when the pointer goes out of scope and no other pointer is pointing to the object). In this case, *openscenegraph* is using `ref_ptr`. Besides, `ref_ptr` is a template so:
```txt
osg::ref_ptr<osg::Geometry>
^^^------------------------- namespace
     ^^^^^^^---------------- smartpointer (template)
	         ^^^^^^^^^^^^^-- type being of the object being "pointed"
			 ^^^------------ namespace
				  ^^^^^^^^-- type
```

How do we wrap this template? By means of:
```nim
type
  RefPtr*[T] {.header: "<osg/Referenced>", importcpp: "osg::ref_ptr<\'0>".} = object
```
where:
- `RefPtr*[T]` will be the type used in Nim (which is a [generic](https://nim-lang.org/docs/tut2.html#generics)).
- `header: "<osg/Referenced>"`: the [header pragma](https://nim-lang.org/docs/manual.html#implementation-specific-pragmas-header-pragma) indicates the name of the header file being wrapped. This will add `#include <osg/Referenced>` in the generated code.
- `importcpp: "osg::ref_ptr<\'0>"`: here we are using the [importcpp pragma](https://nim-lang.org/docs/manual.html#implementation-specific-pragmas-importcpp-pragma). In this case, basically states the it will generate the code `osg::ref_ptr<.....>`  and the `\'0` refers to the content of the first type parameter of the generic (in this case, it refers to the `T`).

> If we were wrapping a template with two types, for example: `xxx::yyy<ppp,qqq>`, the signature would be something like: `Yyy*[P,Q] {. importcpp:"xxx::yyy<\'0,\'1>".}`.

Then, we need to wrap the `osg::Geometry` type. You can do so by means of:
```
type
  GeometryObj {.importcpp: "osg::Geometry",
            header: "<osg/Geometry>", bycopy.} = object
```
where:
- `GeometryObj` will be the equivalent type in Nim
- we can see that we simply refer to the equivalent C++ code: `osg::Geometry`.
- we refer to the header file where that type is defined: `<osg/Geometry>`.
- `bycopy`: the [bycopy pragma](https://nim-lang.org/docs/manual.html#foreign-function-interface-bycopy-pragma) indicates the compiler to pass by value (instead of by reference) instances of this type in procs.

But in C++ we are defining a smart pointer of type `osg::Geometry`. How do we do that in Nim? With above's bricks, this is easy:
```nim
type
  GeometryRef = RefPtr[GeometryObj]
```
where:
- `GeometryRef`: will be the type used in Nim (we call it with a friendly name: reference to Geometry).
- `RefPtr[T]` was a generic, where `T` is the type parameter.
- `GeometryObj` is the type used a parameter for the generic.

So the equivalent to C++'s:
```c++
osg::ref_ptr<osg::Geometry> myTriangleGeometry =
```
in Nim will be:
```nim
var myTriangleGeometry:GeometryRef =
```
