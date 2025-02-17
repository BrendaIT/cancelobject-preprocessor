Klipper GCode Preprocessor for Object Cancellation
==================================================

This preprocessor modifies GCode files to add klipper's exclude object gcode.

The following slicers are supported:

* SuperSlicer
* PrusaSlicer
* Slic3r
* Cura
* IdeaMaker

## Installation and usage

### SuperSlicer, PrusaSlicer, and Slic3r

Download the provided binary for your platform, and place it in with in your slicer's folder.

In your Print Settings, under Output Options, add `preprocessor.exe;` to the "Post-Processing Scripts".
For mac or linux, you should just use `preprocessor;`

Then, all generated gcode should be automatically processed and rewritten to support cancellation.

### G-Codes for Object Cancelation

There are 3 gcodes inserted in the files automatically, and 4 more used to control the 
object cancellation.

`DEFINE_OBJECT NAME=<object name> [CENTER=x,y] [POLYGON=[[x,y],...]]`

The NAME must be unique and consistent throughout the file. CENTER is the center location 
for the mesh, used to show on interfaces where and object being canceled is on the bed. 
POLYGON is a series of points, used to represent the bounds of the object. It can be just 
a bounding box, a simplified outline, or another useful shape.

`START_CURRENT_OBJECT NAME=<object name>` and `END_CURRENT_OBJECT [NAME=<object name>]`

The beginning and end markers for the gcode for a single object. When an object is excluded, 
anything between these markers is ignored.

`LIST_OBJECTS`
`LIST_EXCLUDED_OBJECTS`
`EXCLUDE_OBJECT NAME=<object name>`
`REMOVE_ALL_EXCLUDED`

These gcode are used by the user to locate objects, and control which are canceled. These 
are mostly utility, to be functional without the need for a full frontend for the printer.

### Known Limitations

Cura and Ideamaker sliced files have all support material as a single non-mesh entity.
This means that when canceling an object, it's support will still print. Including
support that is inside or built onto the canceled mesh. The Slic3r family (including
PrusaSlicer and SuperSlicer) treat support as part of the individual mesh's object,
so canceling a mesh cancels it's support as well.

### How does it work

This looks for known markers inside the GCode, specific to each slicer. It uses those
to figure out the printing object's name, and track's all extrusion moves within it's 
print movements. Those are used to calculate a minimal bounding box for each mesh. 
A series of `DEFINE_OBJECT` gcodes are placed in a header, including the bounding boxes
and objects centers. Then, these markers are used to place `START_CURRENT_OBJECT` and 
`END_CURRENT_OBJECT` gcodes in the file surrounding each set of extrusions for that object.
