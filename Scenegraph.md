# Using the QtQuick SceneGraph to render the main display
See [this page](https://github.com/torriem/QtAgOpenGPS/wiki/claude-conversation-about-scenegraph) For a conversation with Claude.ai about different aspects of using the Scene Graph that are germain to this document.

One possibility for rendering the main display in QtAOG is to use QtQuick Scene Graph primitives and turn everything over to the scenegraph rendering pipeline, rather than try to do OpenGL rendering ourselves.  The Scenegraph system is intended for 2D UI primarily, so there is no depth testing of pixels. However QtAOG currently uses essentially 2.5D rendering with no depth testing already.  The field, coverage, implement, tractor, AB lines, flags, etc, are all drawn using a painters algorithm, essentially at a flat z depth of 0.  Essentially all OpenGL drawing is down in 2D on the z=0 plane.

Qt's Scenegraph preserves the 3D Z value throughout the pipeline and uses it to order the final compositing of item renderings into the window. So we do need to ensure that our final result ends up at z=0.  More on this later.

Items in QtQuick are composed of a tree of scenegraph nodes.  We would primarily work with the following node types:

 * QSGNode - a non-rendering node that contains other nodes. Like a folder
 * QSGTransformNode - a node that transforms all its children with a 4x4 matrix.  We can generate this matrix in the same way that oglMainPaint currently does.  If we also create a viewport matrix to multiply the modelview/projection matrix by, we can get from world coordinates to 2D screen coordinates with a single matrix.  Also it's possible to modify the matrix to ensure the final Z value is always zero.  Here's some code to create a viewport matrix;

```cpp
// Viewport: NDC → pixel coordinates
QMatrix4x4 viewport;
viewport.translate(viewportWidth / 2.0f, viewportHeight / 2.0f, 0);
viewport.scale(viewportWidth / 2.0f, -viewportHeight / 2.0f, 1);  // negative Y to flip

QMatrix4x4 vmvp = viewport * modelviewprojection
```
 * GSGGeometryNode - This node contains 2D vertices that make up the shape or shapes. Can be triangles, triangle strips, lines, line strips, and points.  Triangle fans are not supported, but can be emulated with triangles.  Vertices can also have colors, and the final shape interpolates, just like regular OpenGL. Textures are also supported.

# Abstract structure
We can break down our main display into the following general categories, ordered roughly the same as in oglMain_Paint to preserve the way the painter's algorithm was employed in Brian's original work.

 * The underlying field
   * Background texture or just a color.  Could be a downloaded tile set for satellite view
   * Grid
 * Field boundaries, headlands 
 * Patches.  Each color is a different material for batching
 * Guidance lines
 * Flags
 * Vehicle
 * Implement and toolbar, including lookahead lines if turned on

# Possible methodologies
There are different ways in which the scenegraph could be used to render the QtAOG main display, and different ways of organizing the items and nodes.
## Single QQuickItem that builds scene tree for every item update
This is is a single class derived from QQuickItem that builds the node tree from scratch every time updatePaintNode() is called, similar to how oglMain_Paint() operated prior to the OpenGL buffer optimizations.  The scene graph engine will still try to batch as many primitives as it can, grouping them by material. So underlying GPU calls are kept as minimum as possible. However the node structures and the underlying buffers have to be created and filled every time.  In AgOpenGPS this still results in adequate rendering speed, so there's every possibility it will work just fine here.  And this is my recommended strategy for the first attempt.

Pros:

 * Simplest to implement
 * One QQuickItem is required
 * Other than custom updatePaintNode() logic, the rest of QtAOG does not need to know anything about it.
 * No mechanism needed to pass changes to the item from outside of the render thread. No queues, etc

Cons:

 * Potentially slow since the entire node tree has to be built every time, and then passed to GPU buffers for rendering
 * Must use locking when accessing QtAOG structures
 * Requires the UI to have more knowledge of QtAOG internals

## Single QQuickItem with smart, custom QSGNode-derived classes
Another possibility is to make custom Node classes that represent different parts of the display.  For example:

 * FieldNode
   * Sets up the base texture
   * Sets up the grid
 * CoverageNode
   * Holds the nodes that contain the vertices for the coverage patches
   * Contains caching logic to only build new nodes when patches are added
 * etc

This style of system could update these structures based on what is in QtAOG's structures, or QtAOG could tell the QQuickItem what has changed in a proactive manner.

Pros:

 * No need to destroy the node tree and start over each time
 * Things that haven't changed are still in the tree and hopefully still in the underlying GPU buffers
 * More flexible and easier to change
 * Various components that currently implement Draw methods in OpenGL could instead generate or update nodes

Cons:

 * A bit more complexity
 * Need custom QSGNode-derived classes.
 * Some redundancy of storage.  If the classes are to maintain state and only update things that change, then they have to maintain this data in a variable, as well as in the scenegraph node tree.

## Multiple QQuickItems which are layerd in the QML scene
Another possibility is to have multiple QQuickItems that are placed on top of each other in QML.  For example, we might have the following Items:

 * Field - the texture and grid
 * Boundaries and Headlands
 * Coverage
 * Track lines
 * Vehicle and Toolbar
 * Flags

Pros

 * Could support multiple instances of coverage to get layers. For example, yield mapping could be another instance.
 * More flexibility in controlling the display from QML. For example, if there were multiple layers, javascript could turn them on and off.
 * Better-defined interfaces, where each QQuickItem does fewer things. 
Cons

 * More complexity
 * Synchronizing updates is a bit harder. Ideally after a GPS frame comes in we want to update all of these items at once, rather than one at a time, although ultimately it won't matter since the scenegraph engine can rapidly composite the updated textures to the screen, even if that has to happen more than once per GPS update.
 * Again we have the potential for redundant data storage.

Like the single QQuickItem option, this one could also operate either by fetching changes out of QtAOG, or by having QtAOG make calls of the various QQuickItems to update them when changes occur.


