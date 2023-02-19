# Constructors
In this case, a new pointer to an instance of `osg::Geometry` is created in C++ with:
```c++
new osg::Geometry
```
using C++'s [new operator](https://cplusplus.com/reference/new/operator%20new/).

In Nim we can create a constructor as follows:
```nim
proc newGeometryRef*(): GeometryRef {.importcpp: "(new osg::Geometry)", 
                                      header: "<osg/Geometry>".}
```
where we use `importcpp` with the same code we used in C++.

So the equivalent to:
```c++
osg::ref_ptr<osg::Geometry> myTriangleGeometry = new osg::Geometry;
```
will be in Nim:
```nim
var myTriangleGeometry:GeometryRef = newGeometryRef()
```

!!! TODO: cnew, constructor
