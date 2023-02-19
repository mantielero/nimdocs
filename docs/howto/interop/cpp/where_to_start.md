# Where to start
In my experience, the best thing is to start with some easy working example of the library, rather than starting directly with a header.

For example, let's play with [openscenegraph](http://www.openscenegraph.com/):

> OpenSceneGraph is an open source high performance 3D graphics toolkit.

Let's find a basic working example `ex01.cpp`:
```c++
#include <osg/Referenced>
#include <osg/Geometry>
#include <osg/Geode>
#include <osg/Array>
#include <osgViewer/Viewer>

int main(int argc, char** argv)
{
    // Create the geometry which will define the triangle
    osg::ref_ptr<osg::Geometry> myTriangleGeometry = new osg::Geometry;

    // Define the triangle's 3 vertices
    osg::ref_ptr<osg::Vec3Array> vertices = new osg::Vec3Array;
    vertices->push_back(osg::Vec3(0, 0, 0));
    vertices->push_back(osg::Vec3(100, 0, 0));
    vertices->push_back(osg::Vec3(0, 0, 100));
    myTriangleGeometry->setVertexArray(vertices);

    // You can give each vertex its own color, but let's just make it green for now
    osg::ref_ptr<osg::Vec4Array> colors = new osg::Vec4Array;
    colors->push_back(osg::Vec4(0, 1.0, 0, 1.0)); // RGBA for green
    myTriangleGeometry->setColorArray(colors);
    myTriangleGeometry->setColorBinding(osg::Geometry::BIND_OVERALL);

    // Turn off lighting
    myTriangleGeometry->getOrCreateStateSet()->setMode(GL_LIGHTING, osg::StateAttribute::OFF);

    // Turn on blending
    myTriangleGeometry->getOrCreateStateSet()->setMode(GL_BLEND, osg::StateAttribute::ON);

    // Define the geometry type as 'triangles'
    myTriangleGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::TRIANGLES, 0, vertices->size()));

    // Finally, let's add our triangle to a geode
    osg::ref_ptr<osg::Geode> myGeode = new osg::Geode;
    myGeode->addDrawable(myTriangleGeometry);

    // And now we can create a viewer to look at our geode
    osgViewer::Viewer viewer;
    viewer.setSceneData(myGeode);
    return viewer.run();
}
```

I am using ArchLinux and I can compile above's code with:
```sh
g++ -losg -losgViewer -I/usr/include/osg -I/usr/include/osgViewer ex01.cpp
```
