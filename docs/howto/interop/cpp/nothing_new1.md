
### Next C++ line
The following line to wrap:
```c++
osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
```
is not such an alien to us. It follows the same pattern as the first one.

So we will do something similar. Wrap the types and create the constructor:
```nim
type
  Vec3ArrayObj {.importcpp: "osg::Vec3Array",
            header: "<osg/Array>", bycopy.} = object
  Vec3ArrayRef* = RefPtr[Vec3ArrayObj]

proc newVec3ArrayRef*(): Vec3ArrayRef {.cdecl, 
    importcpp: "(new osg::Vec3Array)", header: "<osg/Array>".}
```
where the only difference is on the `header` used.

And we use the newly created types and constructor in Nim:
```nim
var vertices:Vec3ArrayRef = newVec3ArrayRef()
```
