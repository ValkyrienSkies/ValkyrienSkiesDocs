# How to create a custom collision body

<secondary-label ref="experimental"/>

Create a standalone `VSBody` with a collision shape chosen by your addon. This is useful for physics objects that are
not normal block ships, such as tools, machines, temporary gameplay objects, or addon-managed voxel volumes.

This API is currently beta. Names and construction helpers may change before it is promoted to stable.

## Before you start

Make sure that:

- You have [added Valkyrien Skies to your dev environment](How-to-add-Valkyrien-Skies-to-your-dev-environment.md).
- You have access to a server-side `ServerShipWorld`.
- You already know the body's initial inertia data. Primitive shapes do not automatically calculate mass properties.

> Ships still calculate their own block-based mass and inertia. Standalone custom bodies need explicit inertia data
> unless they are using addon-managed voxel segment updates with explicit block masses.
>
{style="note"}

## Create a primitive compound body

Use `VsCoreApi` shape helpers to build a `CompoundBodyShapeData`, then pass it to `ServerShipWorld.createBody`.

<code-block lang="Kotlin">
import org.joml.Quaterniond
import org.joml.Vector3d
import org.valkyrienskies.core.api.bodies.VsBodyCreateData
import org.valkyrienskies.core.api.bodies.properties.BodyInertia
import org.valkyrienskies.core.api.world.ServerShipWorld
import org.valkyrienskies.mod.api.ValkyrienSkies

val vsCore = ValkyrienSkies.api
val shipWorld: ServerShipWorld = ...
val inertiaData: BodyInertia = ...

val compoundShape = vsCore.newCompoundBodyShape(
    listOf(
        vsCore.newCompoundBodyShapeChild(
            shape = vsCore.newBoxBodyShape(Vector3d(1.0, 1.0, 1.0)),
            position = Vector3d(1.5, 0.0, 0.0),
        ),
        vsCore.newCompoundBodyShapeChild(
            shape = vsCore.newCapsuleBodyShape(radius = 0.25, halfLength = 1.0),
            position = Vector3d(-1.5, 0.0, 0.0),
            rotation = Quaterniond().rotateZ(Math.PI / 2.0),
        ),
    )
)

val body = shipWorld.createBody(
    VsBodyCreateData(
        dimensionId = "overworld",
        inertiaData = inertiaData,
        kinematics = vsCore.newBodyKinematics(
            velocity = Vector3d(),
            angularVelocity = Vector3d(),
            position = Vector3d(10.0, 64.0, 10.0),
            rotation = Quaterniond(),
            scaling = Vector3d(1.0),
            positionInModel = Vector3d(),
        ),
        collisionShape = compoundShape,
        isStatic = false,
    )
)
</code-block>

Each child has its own body-local transform. A capsule can be rotated separately from a box even though both children
belong to the same rigid body.

## Use segment IDs

VS-Core assigns segment IDs when you call `newCompoundBodyShape`. Do not invent segment IDs in addon code.

<code-block lang="Kotlin">
val capsuleSegmentId = compoundShape.children[1].segmentId

body.setCollisionSegmentTransform(
    segmentId = capsuleSegmentId,
    position = Vector3d(-2.0, 0.0, 0.0),
    rotation = Quaterniond().rotateX(0.5),
)
</code-block>

Segment IDs are local to one body. They are not global, but they are also not reused for the same body after a segment is
removed. This prevents stale physics updates or contact data from accidentally targeting a newer segment.

## Add and remove segments

Use `addCollisionSegment` when a body already exists and you need to attach another child shape.

<code-block lang="Kotlin">
val sphereSegmentId = body.addCollisionSegment(
    shape = vsCore.newSphereBodyShape(radius = 0.75),
    position = Vector3d(0.0, 2.0, 0.0),
    rotation = Quaterniond(),
)

body.removeCollisionSegment(sphereSegmentId)
</code-block>

The returned segment ID is the only ID you should use for later updates to that segment.

## Add a voxel segment

Voxel segments let an addon manage a block-like collision volume inside a compound body.

<code-block lang="Kotlin">
import org.joml.Vector3i
import org.joml.primitives.AABBi

val voxelSegmentId = body.addCollisionSegment(
    shape = vsCore.newVoxelBodyShape(
        minDefined = Vector3i(0, 0, 0),
        maxDefined = Vector3i(15, 15, 15),
        totalVoxelRegion = AABBi(0, 0, 0, 15, 15, 15),
        voxelsFullyLoaded = true,
    ),
    position = Vector3d(3.0, 0.0, 0.0),
    rotation = Quaterniond(),
)
</code-block>

`minDefined` and `maxDefined` describe the defined voxel bounds in the segment's local voxel coordinates. The default
shape uses one 16 by 16 by 16 chunk, but larger regions can be represented with multiple voxel updates.

## Update voxel contents

Build a `VoxelUpdate` and apply it to the voxel segment ID returned by `addCollisionSegment`.

<code-block lang="Kotlin">
import org.valkyrienskies.core.api.bodies.shape.VoxelType

val blockType: VoxelType = ...

val update = vsCore.newSparseVoxelUpdateBuilder(chunkX = 0, chunkY = 0, chunkZ = 0).apply {
    addBlock(x = 4, y = 5, z = 6, block = blockType, mass = 2.0)
}.build()

body.applyVoxelSegmentUpdate(voxelSegmentId, update)
</code-block>

Use the explicit `mass` overload when the voxel segment should contribute to the body's mass, center of mass, and
inertia. If you call `addBlock(x, y, z, block)` without a mass, VS uses `VoxelType.mass`; the default value is zero.

## Current limitations

- This API is beta and should be guarded behind your own compatibility layer if your addon needs long-term stability.
- Compound bodies currently support primitive shapes and voxel child shapes.
- Arbitrary triangle meshes, convex hulls, and procedural collision callbacks are not part of the compound-shape API yet.
- Segment updates are routed through the normal game-frame to physics-frame pipeline. Do not expect changes to be visible
  inside the same physics step in which you call the API.
