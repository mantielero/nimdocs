
### Wrapping some procs
After that, we use some methods in C++:
```c++
vertices->push_back(osg::Vec3(0, 0, 0));
```

We have `vertices`'s type already wrapped: `Vec3ArrayRef`. 

The method `push_back` accepts and object typed `osg::Vec3` as first parameter. So we start wrapping that type and constructor:
```nim
type
  Vec3Obj {.importcpp: "osg::Vec3",
            header: "<osg/Array>", bycopy.} = object
			
proc newVec3*(x,y,z:cdouble): Vec3Obj {.
    importcpp: "osg::Vec3(@)", header: "<osg/Array>", constructor.}
```
Nothing to comment about `Vec3Obj` type, but a couple of things about the `newVec3` constructor:
- `osg::Vec3(@)`: here `@` means all the parameters of the `newVec3` proc. So '`@` will be replaced with `x`, `y`, `z`.
- `constructor`:
- Also note: [cdouble](https://nim-lang.org/docs/system.html#cdouble)

[more about constructors](https://nim-lang.org/docs/manual.html#importcpp-pragma-wrapping-constructors)

> TIP: I suggest compiling the nim code with `--nimcache:myfolder`. This way, you can inspect the generated `.cpp` files and see if what is being generated looks like it should. So execute:
> `nim cpp --nimcache:myfolder ex01.nim`

After that, we can wrap the method `push_back`:
```nim
proc pushBack*(arr:Vec3ArrayRef; v:Vec3Obj)  {.cdecl, 
    importcpp: "#->push_back(#)", header: "<osg/Array>".}
```
Here we are using the parameters with types `Vec3ArrayRef` and `Vec3Obj`. It is worth noting:
- `importcpp: "#->push_back(#)"`: where `#` represents the parameters in order; so the first `#` appearance represent the first parameter in the proc (`arr:Vec3ArrayRef`) and the second `#` represent the second parameter in the proc (`v:Vec3Obj`).
> Note: we could use `@` representing the remaining parameters in the proc when we have used `#` before. So above's would be equivalent to :  `importcpp: "#->push_back(@)"`, but `@` would represent the second, third, ... or whatever remaining proc parameters.
- `->`: also note that here we are using `->` insted of a `.` to call the method. This is plain C++ code. In C++, the dot operator is applied to the actual object while the arrow operator is used with a pointer to an object (which is our case now).

We would use the wrapped method as follows:
```nim
vertices.pushBack( newVec3(0.0,0.0,0.0) )
vertices.pushBack( newVec3(100.0,0.0,0.0) )
vertices.pushBack( newVec3(0.0,0.0,100.0) )
```
