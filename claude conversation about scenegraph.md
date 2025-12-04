# QSGTransformNode matrix z-value rendering

**human** (Dec 3, 2025, 06:40 PM)

QSGTransformNode can use a 4x4 matrix.  Does this ultimately discard the z value so everything renders in 2D?

**assistant** (Dec 3, 2025, 06:41 PM)

 QSGTransformNode in Qt Quick's scene graph uses a 4x4 matrix (QMatrix4x4) for transformations, but no, it doesn't simply discard the Z value — the Z coordinate remains meaningful throughout the rendering pipeline.

Here's what actually happens:

1. **The scene graph is 2.5D**: Qt Quick items are conceptually 2D, but the underlying scene graph and renderer work in 3D space. The Z value affects depth ordering and can influence rendering when using perspective projections.

2. **Z affects layering and depth testing**: When you apply transformations that modify Z (like rotations around X or Y axes), items can appear to move in 3D space. The Z value determines which items appear in front of others when depth testing is enabled.

3. **Projection flattens to 2D at the end**: The final projection matrix (typically orthographic for standard Qt Quick, but can be perspective) projects the 3D coordinates onto the 2D screen. This is where the 3D scene becomes a 2D image, but Z information is used for depth sorting before that projection.

4. **With perspective transforms**: If you use something like `transform: Matrix4x4 { ... }` with perspective, or set a custom projection on the scene graph, Z differences become visible as foreshortening effects.

So the full 4x4 matrix is used throughout the scene graph traversal and batching. The "flattening" only occurs during final projection to screen coordinates, and even then Z is used for proper depth ordering.

**human** (Dec 3, 2025, 06:41 PM)

If the QSGGeometryNode is using 2D vertices, does that mean the final result will always be z=0 after being transformed?

**assistant** (Dec 3, 2025, 06:42 PM)

 Not necessarily. It depends on the transformation matrix you apply.

When your QSGGeometryNode uses 2D vertices (like `QSGGeometry::Point2D`), those vertices implicitly have z=0. But when the 4x4 matrix transforms them, the resulting Z depends on the matrix contents.

For a vertex at (x, y, 0, 1), multiplying by a 4x4 matrix gives:

```
| m11 m12 m13 m14 |   | x |   | m11*x + m12*y + m14 |
| m21 m22 m23 m24 | × | y | = | m21*x + m22*y + m24 |
| m31 m32 m33 m34 |   | 0 |   | m31*x + m32*y + m34 |
| m41 m42 m43 m44 |   | 1 |   | m41*x + m42*y + m44 |
```

The resulting Z component is `m31*x + m32*y + m34`. So:

- **Simple 2D transforms** (translation, scale, 2D rotation) leave m31, m32, and m34 at zero, so Z stays at 0.

- **3D rotations** around X or Y axes populate m31 and m32, meaning your originally-flat vertices will end up with varying Z values based on their X/Y positions. A rectangle rotated around the Y axis will have its left and right edges at different Z depths.

- **Z translation** (m34 non-zero) shifts all vertices to a uniform non-zero Z.

So 2D input vertices don't guarantee z=0 output — the matrix determines that. The 2D vertex format just means you're starting from a flat plane at z=0 before transformation.

**human** (Dec 3, 2025, 06:42 PM)

Is there any way to make sure the scenegraph flattens the result before it tries to composite the results together?

**assistant** (Dec 3, 2025, 06:42 PM)

 Not directly — Qt's scene graph doesn't have a built-in "flatten Z before compositing" option. But you have a few approaches depending on what you're trying to achieve:

**1. Flatten in your transform matrix**

Multiply your transformation matrix by a projection matrix that zeroes out Z before setting it on the QSGTransformNode. You can create a matrix that preserves X and Y but sets Z to a constant:

```cpp
QMatrix4x4 flatten;
flatten(2, 0) = 0;
flatten(2, 1) = 0;
flatten(2, 2) = 0;
flatten(2, 3) = 0; // or some constant Z value

QMatrix4x4 finalMatrix = flatten * yourTransform;
transformNode->setMatrix(finalMatrix);
```

This loses the 3D effect though, which may defeat the purpose.

**2. Use layers (ShaderEffectSource)**

Wrap your 3D-transformed item in a `layer.enabled: true` or `ShaderEffectSource`. This renders the item to a texture first (flattening it), then composites that texture as a flat quad:

```qml
Item {
    layer.enabled: true
    transform: Matrix4x4 { /* your 3D transform */ }
    // children render to texture, then texture is composited flat
}
```

**3. Custom QSGRenderNode**

If you need precise control, implement a QSGRenderNode where you manage the rendering yourself and explicitly set the projection/modelview to flatten before the draw call.

**4. Disable depth testing**

This doesn't flatten geometry, but if your issue is depth-based sorting conflicts, you can ensure items composite in painter's order by keeping depth testing off (which is the default for Qt Quick).

What's the underlying problem you're trying to solve? That might help narrow down the best approach.

**human** (Dec 3, 2025, 06:44 PM)

How do I disable depth testing? Is this for just one QQuickItem or is it for the entire scene?

**assistant** (Dec 3, 2025, 06:44 PM)

 Depth testing is off by default for the standard Qt Quick scene graph, so if you're seeing depth-related issues, something is enabling it.

**For the entire scene**, you can control this via the render loop. In Qt 6 with RHI, you can set up a QQuickGraphicsConfiguration, but depth testing control is more indirect there.

**For individual items**, you have a couple options:

**1. QSGRenderNode**

This gives you the most control. Override the `flags()` method:

```cpp
QSGRenderNode::RenderingFlags MyNode::flags() const
{
    return QSGRenderNode::DepthAwareRendering; // or omit this flag to ignore depth
}
```

And in your `render()` implementation, you can explicitly set the depth state however you want using the graphics API directly.

**2. QSGMaterialShader with custom state**

If you're using a custom material, you can override `updateGraphicsPipelineState()` in Qt 6:

```cpp
bool MyShader::updateGraphicsPipelineState(RenderState &state, 
                                            GraphicsPipelineState *ps,
                                            GraphicsPipelineState::DirtyStates dirty)
{
    ps->depthTestEnable = false;
    ps->depthWriteEnable = false;
    return true;
}
```

**3. Check if something else is enabling it**

If you're using `layer.enabled` with `layer.mipmap` or certain ShaderEffect configurations, or if you've set `QSG_NO_DEPTH_BUFFER=0` or similar environment variables, depth testing might be getting enabled. Also check if you're using any Qt3D integration or custom render loops.

What renderer backend are you using (OpenGL, Vulkan, Metal, D3D)? And are you seeing specific visual artifacts that suggest depth testing is on?

**human** (Dec 3, 2025, 06:51 PM)

In a QQuickItem, does the scengraph render the node tree top down?

**assistant** (Dec 3, 2025, 06:51 PM)

 Yes, the scene graph renders in a top-down, depth-first order by default. For a node tree like:

```
Root
├── Node A
│   ├── Child A1
│   └── Child A2
└── Node B
```

The render order is: Root → Node A → Child A1 → Child A2 → Node B.

This gives you painter's algorithm ordering — earlier nodes get drawn first and later nodes paint over them.

However, there are some nuances:

**Batching complicates this**: The Qt Quick renderer batches geometry for performance. It groups nodes with similar materials together and may reorder draws within those batches. For opaque items this is fine, but for translucent items the renderer tries to preserve the correct back-to-front order.

**You can influence order with QSGNode flags**:

```cpp
node->setFlag(QSGNode::OwnedByParent);
// Insert order matters:
parentNode->appendChildNode(drawFirst);
parentNode->appendChildNode(drawSecond);
```

**Z value in Items affects QML ordering**: In QML, the `z` property on Items controls sibling order, which translates to node order in the scene graph. Higher `z` values render later (on top).

**QSGRenderNode is special**: If you use QSGRenderNode, you can specify ordering hints via `flags()` and `changedStates()` to tell the renderer how your custom node interacts with batching.

So while the traversal is top-down, the actual draw call order depends on how the renderer batches things. If you need strict ordering, you may need to break batching by using different materials or layers.

**human** (Dec 3, 2025, 06:53 PM)

Can a sub-classed node simply be a container that itself is not rendered?

**assistant** (Dec 3, 2025, 06:53 PM)

 Yes, absolutely. Just subclass QSGNode directly rather than QSGGeometryNode or QSGRenderNode.

```cpp
class ContainerNode : public QSGNode
{
public:
    ContainerNode()
    {
        // Add children as needed
        appendChildNode(new SomeGeometryNode());
        appendChildNode(new AnotherGeometryNode());
    }
};
```

QSGNode is the base class and has no rendering of its own — it's purely a structural node for organizing the tree. The renderer traverses it but draws nothing for it directly.

QSGTransformNode and QSGOpacityNode work the same way — they're containers that affect their children but don't render anything themselves. They just apply a transform or opacity to the subtree beneath them.

This is useful for:

- Grouping related nodes logically
- Applying transforms or opacity to a group via QSGTransformNode/QSGOpacityNode
- Building reusable composite node structures
- Managing node lifetimes together

One thing to note: if your container doesn't add any meaningful structure (no transform, no opacity, no clipping), you could also just append children directly to the parent. But having a container node makes the code cleaner and lets you add/remove the whole group as a unit.

**human** (Dec 3, 2025, 07:40 PM)

Does QSGTransformNode apply to all its children nodes?  In other words, should I put this node as the first node in a QQuickItem if I want it to apply to everything drawn by that item?

**assistant** (Dec 3, 2025, 07:40 PM)

 Yes, QSGTransformNode applies its matrix to all its descendants. So if you want it to affect everything in your QQuickItem, it should be the root node returned by `updatePaintNode()`:

```cpp
QSGNode* MyItem::updatePaintNode(QSGNode *oldNode, UpdatePaintNodeData *)
{
    QSGTransformNode *transformNode = static_cast<QSGTransformNode*>(oldNode);
    if (!transformNode) {
        transformNode = new QSGTransformNode();
        transformNode->appendChildNode(new SomeGeometryNode());
        transformNode->appendChildNode(new AnotherGeometryNode());
    }
    
    transformNode->setMatrix(myMatrix);
    return transformNode;
}
```

The node you return becomes the root of that item's subtree in the scene graph. Everything beneath it inherits the transform.

Transforms also accumulate down the tree. If you nest transform nodes:

```
QSGTransformNode (rotate 45°)
└── QSGTransformNode (scale 2x)
    └── QSGGeometryNode
```

The geometry node gets both transforms applied — scaled then rotated (or rather, the matrices multiply together as you traverse down).

One thing to keep in mind: the QQuickItem's own transform (from QML `transform`, `rotation`, `scale`, `x`, `y` properties) is applied above your node tree by Qt. So your returned node is already a child of Qt's internal transform node for that item. If you want your QSGTransformNode to apply additional transformation beyond what the item properties provide, that works fine. But you don't need to replicate the item's basic positioning — Qt handles that.

**human** (Dec 3, 2025, 07:41 PM)

If a QSGTransformNode is used, and if any children nodes end up being off the screen, will they be culled before rendered, or will they be rendered anyway?

**assistant** (Dec 3, 2025, 07:41 PM)

 The Qt Quick scene graph does not do automatic frustum culling based on transformed geometry. Nodes that end up off-screen after transformation will still be processed and their vertices sent to the GPU.

The renderer does do some culling, but it's based on the original bounding rectangles of QQuickItems before transformation, not the transformed geometry. So if your item's bounding rect intersects the visible area, all its nodes get rendered regardless of where the transform puts them.

This means:

**No automatic culling**: If you have a large node tree and transform most of it off-screen, you're still paying the cost of batching and issuing draw calls for that geometry. The GPU will clip it, but the CPU-side work still happens.

**You can cull manually**: If performance matters, you can do your own culling in `updatePaintNode()`:

```cpp
QSGNode* MyItem::updatePaintNode(QSGNode *oldNode, UpdatePaintNodeData *)
{
    // ...
    
    for (each child) {
        QRectF transformedBounds = myMatrix.mapRect(childBounds);
        if (transformedBounds.intersects(visibleArea)) {
            // Add or keep the child node
        } else {
            // Remove or skip the child node
        }
    }
    
    return rootNode;
}
```

**Item-level culling exists**: Qt does cull entire QQuickItems that are flagged with `ItemHasContents` and whose untransformed bounds don't intersect the visible region. But this happens before your transform node is considered.

So if you're transforming large amounts of geometry off-screen and seeing performance issues, manual culling in your update logic is the way to go.

**human** (Dec 3, 2025, 07:42 PM)

Can I turn off rendering of a particular child node with a flag?

**assistant** (Dec 3, 2025, 07:43 PM)

 There's no built-in "visible" or "enabled" flag on QSGNode to skip rendering while keeping it in the tree.

Your options are:

**1. Remove and re-add the node**

```cpp
parentNode->removeChildNode(childNode);
// Later:
parentNode->appendChildNode(childNode);
```

This is the most common approach. You can keep a pointer to the node and just detach it from the tree when you don't want it rendered.

**2. Use QSGOpacityNode with opacity 0**

```cpp
opacityNode->setOpacity(0.0);
```

The renderer skips nodes with zero opacity. This is a simple toggle if you already have the node wrapped in an opacity node. However, I'm not 100% certain the renderer optimizes this away entirely versus just rendering invisible geometry.

**3. Set an empty or degenerate state on the node**

For a QSGGeometryNode, you could set the vertex count to zero or set a null material, but this is hacky and error-prone.

**4. Track visibility yourself and rebuild**

Keep a flag in your own data structure and simply don't create/attach the node in `updatePaintNode()` when it shouldn't be visible.

The cleanest approach is usually option 1 — just remove the node from the tree. The scene graph is designed for nodes to be added and removed dynamically. Just be mindful of ownership; if you set `OwnedByParent`, removing the node will delete it. If you want to reattach later, manage the lifetime yourself.

**human** (Dec 3, 2025, 07:45 PM)

If the transform is applied at the Item level in QML, is it also true that culling of objects won't occur either?

**assistant** (Dec 3, 2025, 07:45 PM)

 Correct. The same limitation applies.

Qt Quick's culling logic uses the item's untransformed bounding rectangle. When you apply transforms at the QML level via `transform`, `rotation`, `scale`, etc., the culling decision has already been made based on the original bounds.

So if you have:

```qml
Item {
    x: 0; y: 0
    width: 100; height: 100
    
    transform: Matrix4x4 {
        matrix: Qt.matrix4x4(...)  // moves content way off screen
    }
    
    Rectangle { ... }
}
```

The item's bounds (0,0,100,100) are checked against the viewport. If that rectangle is on-screen, the item renders — even if the transform pushes all the actual content off-screen.

The reverse is also problematic: if the untransformed bounds are off-screen but the transform would bring content into view, the item gets culled and you see nothing.

For items with significant transforms, you can work around this by:

**1. Making the item bounds large enough to encompass all possible transformed positions**

```qml
Item {
    x: -500; y: -500
    width: 1000; height: 1000
    // Now culling is less likely to incorrectly exclude it
}
```

**2. Setting `clip: false`** (which is the default, but worth checking)

**3. Disabling culling for the item** by ensuring its bounds always intersect the viewport

It's a known limitation of how Qt Quick optimizes rendering. The scene graph prioritizes batching efficiency over precise per-item culling with transforms applied.

**human** (Dec 3, 2025, 07:45 PM)

Can I set an arbitrary bounding rectangle for my object in pre-transformed coordinates then?

**assistant** (Dec 3, 2025, 07:46 PM)

 Yes. In your QQuickItem subclass, override `boundingRect()`:

```cpp
QRectF MyItem::boundingRect() const
{
    return QRectF(-500, -500, 1000, 1000);
}
```

Or if you want it dynamic based on your transform:

```cpp
QRectF MyItem::boundingRect() const
{
    // Calculate bounds that encompass all possible transformed content
    return m_myTransform.mapRect(m_contentBounds);
}
```

You should also make sure the item's `width` and `height` properties reflect this if other QML code depends on them, or keep them separate and just use `boundingRect()` for culling purposes.

One caveat: `boundingRect()` is used for more than just culling — it affects input hit testing and other things. If you inflate it significantly, you might get unexpected behavior with mouse events. If that's an issue, you can override `contains()` separately to define a tighter hit-test region:

```cpp
bool MyItem::contains(const QPointF &point) const
{
    // Tighter hit testing even with inflated boundingRect
    return m_actualContentBounds.contains(point);
}
```

In QML, you can't override `boundingRect()` directly, but you can achieve a similar effect by wrapping your content in a larger transparent Item that has the bounds you need.

**human** (Dec 3, 2025, 07:46 PM)

So will any node that is entirely outside this bounding rectangle not be rendered (culled)?

**assistant** (Dec 3, 2025, 07:46 PM)

 No, unfortunately not. The bounding rectangle controls whether the entire QQuickItem is culled, not individual nodes within it.

The culling granularity is at the item level:

- If the item's bounding rect is outside the viewport → entire item is culled, none of its nodes render
- If the item's bounding rect intersects the viewport → entire item renders, all of its nodes are processed

There's no per-node culling based on the item's bounding rect. The scene graph doesn't check each QSGGeometryNode against any bounds — once the item passes the culling test, all its nodes go through batching and get submitted to the GPU.

So if you have one QQuickItem with many child nodes spread across a large area, and only some are visible:

```
QQuickItem (huge bounding rect)
├── QSGGeometryNode (on screen)
├── QSGGeometryNode (off screen)
├── QSGGeometryNode (off screen)
└── QSGGeometryNode (on screen)
```

All four nodes get rendered. The GPU clips the off-screen ones, but the CPU work is already done.

If you need per-node culling, you have to do it yourself in `updatePaintNode()` by checking each node's bounds against the visible area and removing/adding nodes from the tree accordingly. Or split your content into multiple QQuickItems so Qt's item-level culling can work on smaller pieces.

**human** (Dec 3, 2025, 08:24 PM)

Can a QQuickItem expose a method that can be called from the the main thread (perhaps during a callback) that adds new nodes to the item?

**assistant** (Dec 3, 2025, 08:24 PM)

 Not safely, no. Scene graph nodes must only be created and manipulated during the `updatePaintNode()` call, which runs on the render thread (or the main thread with the threaded render loop disabled, but synchronized with rendering).

The scene graph documentation is strict about this: touching nodes outside of `updatePaintNode()` leads to race conditions and crashes.

The standard pattern is:

**1. Store data on the main thread, build nodes on the render thread**

```cpp
// Main thread - called from QML or wherever
void MyItem::addShape(const QRectF &rect)
{
    m_pendingShapes.append(rect);
    update();  // Triggers updatePaintNode() on render thread
}

// Render thread
QSGNode* MyItem::updatePaintNode(QSGNode *oldNode, UpdatePaintNodeData *)
{
    // Now it's safe to create/modify nodes
    for (const QRectF &rect : m_pendingShapes) {
        auto *node = new QSGGeometryNode();
        // ... set up node ...
        rootNode->appendChildNode(node);
    }
    m_pendingShapes.clear();
    return rootNode;
}
```

**2. Use `QQuickItem::update()` to trigger a repaint**

This schedules `updatePaintNode()` to be called on the next frame.

**3. Protect shared data if needed**

If your callback might fire while rendering is happening, use a mutex around the shared data:

```cpp
void MyItem::addShape(const QRectF &rect)
{
    QMutexLocker locker(&m_mutex);
    m_pendingShapes.append(rect);
    update();
}
```

So your main-thread method queues up the work, and `updatePaintNode()` actually builds the nodes.
