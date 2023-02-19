
# Passing parameters to the compiler
We will start with the parameters that the compiler requires in order to get the code compiled. In the compilation process, we used `-losg -losgViewer -I/usr/include/osg -I/usr/include/osgViewer`.

Our Nim file `ex01.nim` would start in this case with:
```nim
{.passL: "-losg -losgViewer".}
{.passC: "-I/usr/include/osg -I/usr/include/osgViewer"}
```

As the manual states, the [passL pragma](https://nim-lang.org/docs/manual.html#implementation-specific-pragmas-passl-pragma) passes parameters to the linker.

You can also use `pkgconfig`:
```sh
$ pkg-config --libs openscenegraph
-losg -losgDB -losgFX -losgGA -losgParticle -losgSim -losgText -losgUtil -losgTerrain -losgManipulator -losgViewer -losgWidget -losgShadow -losgAnimation -losgVolume -lOpenThreads
```

The way in which the outcome of `pkgconfig` is used in `passL` is by means of [gorge](https://nim-lang.org/docs/system.html#gorge%2Cstring%2Cstring%2Cstring):
```nim
{.passL: gorge("pkgconfig --libs openscenegraph").}
```

The same applies to [passC](https://nim-lang.org/docs/manual.html#implementation-specific-pragmas-passc-pragma). So you could use:
```nim
{passC: gorge("pkg-config --cflags openscenegraph")}
```

So we can live with:
```nim
{.passL: gorge("pkgconfig --libs openscenegraph").}
{passC: gorge("pkg-config --cflags openscenegraph")}  # In this case will be empty
```

!!! TODO: windows case
