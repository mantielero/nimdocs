
### Next lines
Nothing special about the next ones:
```c++
myTriangleGeometry->setVertexArray(vertices);

// You can give each vertex its own color, but let's just make it green for now
osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
colors->push_back(osg::Vec4(0, 1.0, 0, 1.0)); // RGBA for green
myTriangleGeometry->setColorArray(colors);
```
will be wrapped like:
```nim
proc setVertexArray*(geom:GeometryRef; vertices:Vec3ArrayRef)  {.cdecl, 
    importcpp: "#->setVertexArray(#)", header: "<osg/Array>".}

type
  Vec4ArrayObj {.importcpp: "osg::Vec4Array",
            header: "<osg/Array>", bycopy.} = object
  Vec4ArrayRef = RefPtr[Vec4ArrayObj]

  Vec4Obj {.importcpp: "osg::Vec4",
            header: "<osg/Array>", bycopy.} = object

proc newVec4ArrayRef*(): Vec4ArrayRef {.cdecl,
    importcpp: "(new osg::Vec4Array)", header: "<osg/Array>".}

proc newVec4*(a,b,c,d:cdouble): Vec4Obj {.
    importcpp: "osg::Vec4(@)", header: "<osg/Array>", constructor.}
	
proc pushBack*(arr:Vec4ArrayRef; v:Vec4Obj)  {.cdecl, 
    importcpp: "#->push_back(#)", header: "<osg/Array>".}

proc setColorArray*(geom:GeometryRef; colors:Vec4ArrayRef)  {.cdecl, 
    importcpp: "#->setColorArray(#)", header: "<osg/Array>".}
```

So the code will be:
```nim
  myTriangleGeometry.setVertexArray( vertices )

  # You can give each vertex its own color, but let's just make it green for now
  # osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
  var colors = newVec4ArrayRef()
  # colors->push_back(osg::Vec4(0, 1.0, 0, 1.0)); // RGBA for green
  colors.pushBack( newVec4(0.0,1.0,0.0,1.0) ) # RGBA for green
  # myTriangleGeometry->setColorArray(colors);
  myTriangleGeometry.setColorArray(colors)
```
### Next lines
Nothing special about the next ones:
```c++
myTriangleGeometry->setVertexArray(vertices);

// You can give each vertex its own color, but let's just make it green for now
osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
colors->push_back(osg::Vec4(0, 1.0, 0, 1.0)); // RGBA for green
myTriangleGeometry->setColorArray(colors);
```
will be wrapped like:
```nim
proc setVertexArray*(geom:GeometryRef; vertices:Vec3ArrayRef)  {.cdecl, 
    importcpp: "#->setVertexArray(#)", header: "<osg/Array>".}

type
  Vec4ArrayObj {.importcpp: "osg::Vec4Array",
            header: "<osg/Array>", bycopy.} = object
  Vec4ArrayRef = RefPtr[Vec4ArrayObj]

  Vec4Obj {.importcpp: "osg::Vec4",
            header: "<osg/Array>", bycopy.} = object

proc newVec4ArrayRef*(): Vec4ArrayRef {.cdecl,
    importcpp: "(new osg::Vec4Array)", header: "<osg/Array>".}

proc newVec4*(a,b,c,d:cdouble): Vec4Obj {.
    importcpp: "osg::Vec4(@)", header: "<osg/Array>", constructor.}
	
proc pushBack*(arr:Vec4ArrayRef; v:Vec4Obj)  {.cdecl, 
    importcpp: "#->push_back(#)", header: "<osg/Array>".}

proc setColorArray*(geom:GeometryRef; colors:Vec4ArrayRef)  {.cdecl, 
    importcpp: "#->setColorArray(#)", header: "<osg/Array>".}
```

So the code will be:
```nim
  myTriangleGeometry.setVertexArray( vertices )

  # You can give each vertex its own color, but let's just make it green for now
  # osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
  var colors = newVec4ArrayRef()
  # colors->push_back(osg::Vec4(0, 1.0, 0, 1.0)); // RGBA for green
  colors.pushBack( newVec4(0.0,1.0,0.0,1.0) ) # RGBA for green
  # myTriangleGeometry->setColorArray(colors);
  myTriangleGeometry.setColorArray(colors)
```