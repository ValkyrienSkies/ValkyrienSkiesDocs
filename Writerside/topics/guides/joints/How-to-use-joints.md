# How to use joints

Learn how to create, update, and remove joints between two ships.

## Introduction

Joints can be used to connect two ships (or a ship and the world) 
with a variety of connection types including fixed rotation, position, specific locked axis, or other setups.

If you want to create something like a rope or a bearing, then you're in the right place.

> `VSJointId` is a kotlin type alias for an integer. From java, simply use an `Integer` instead.
>
{style="note"}

> GTPA is a shortened name for the `GameToPhysicsAdapter` class, 
> which is used to do many physics-tick only operations from the gametick.
>
{style="note"}

## Steps

### Creating a joint

> If you want to use the world as one of the bodies, you can simply pass `null` as the bodyId.
>
{style="tip"}

To create a joint, you will need:
- Two body ids (which are usually obtained from a ship)
- An instance of a subclass of VSJoint
- Two VSJointPose's to use in constructing the VSJoint

<deflist collapsible="true">
    <def title="Available VSJoint subclasses" default-state="collapsed">
        <code>VSFixedJoint</code>, 
        <code>VSDistanceJoint</code>, 
        <code>VSPrismaticJoint</code>, 
        <code>VSSphericalJoint</code>, 
        <code>VSRevoluteJoint</code>, 
        <code>VSGearJoint</code>, 
        <code>VSRackAndPinionJoint</code>, 
        <code>VSD6Joint</code>
        <br>
        <br>
        Different joint subclasses require different amounts of parameters to be specified. 
        <br>
        <br>
        For simplicity, this page will demonstrate using the <code>VSFixedJoint</code>, which requires the least parameters to set up.
    </def>
</deflist>

Joints need to be created on the physics tick, however the GTPA has methods to allow creation of joints on the game tick.

Here is an example function (using the GTPA) which adds a fixed joint between two ships (at their center of mass, with their current rotation):

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
fun makeJoint(level: Level, ship0: Ship, ship1: Ship) {
    val pose0 = VSJointPose(
        ship0.shipAABB?.center(Vector3d()) ?: Vector3d(),
        ship0.transform.rotation
    )

    val pose1 = VSJointPose(
        ship1.shipAABB?.center(Vector3d()) ?: Vector3d(),
        ship1.transform.rotation
    )

    val gtpa = getOrCreateGTPA(level.dimensionId)
    val joint = VSFixedJoint(ship0.id, pose0, ship1.id, pose1)
    gtpa.addJoint(joint, 0) {}
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public void makeJoint(Level level, Ship ship0, Ship ship1) {
    VSJointPose pose0 = new VSJointPose(
            ship0.getShipAABB().center(new Vector3d()),
            ship0.getTransform().getRotation()
    );

    VSJointPose pose1 = new VSJointPose(
            ship1.getShipAABB().center(new Vector3d()),
            ship1.getTransform().getRotation()
    );

    GameToPhysicsAdapter gtpa = ValkyrienSkiesMod.getOrCreateGTPA(
        VSGameUtilsKt.getDimensionId(level)
    );
    VSFixedJoint joint = new VSFixedJoint(ship0.getId(), pose0, ship1.getId(), pose1, null, VSJoint.DEFAULT_COMPLIANCE);
    gtpa.addJoint(joint, 0, (id) -> {});
}
</code-block>
</tab>
</tabs>

If you are looking to add a joint on the physics tick, look at how the [`GameToPhysicsAdapter`](https://github.com/ValkyrienSkies/Valkyrien-Skies-2/blob/1.20.1/main/common/src/main/kotlin/org/valkyrienskies/mod/common/util/GameToPhysicsAdapter.kt) does it.

### Storing the joint id

If you are keen-eyed, you will have noticed the empty lambda we passed into the addJoint method in the previous code example.

This lambda (which is a `VSJointId` consumer) will be run on the physics tick once the joint has been added. 
The `VSJointId` the lambda provides you is the (integer) id which has been allocated to your joint. 

Addons will often use this to store the id into a variable in a block entity for updating/removing the joint.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
private var jointId: VSJointId? = null

fun makeJoint(level: Level, ship0: Ship, ship1: Ship) {
    // ...
    gtpa.addJoint(joint, 0) { id ->
        jointId = id
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
private @Nullable Integer jointId = null;

public void makeJoint(Level level, Ship ship0, Ship ship1) {
    // ...
    gtpa.addJoint(joint, 0, (id) -> {
        jointId = id;
    });
}
</code-block>
</tab>
</tabs>

### Updating the joint

To update an existing joint you can get the current joint, copy its properties to a new `VSJoint` instance and change as needed. 
Then construct an instance of `VSJointAndId` to give to the GTPA to update.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
fun updateJoint(level: Level, newPos0: Vector3d, jointId: VSJointId) {
    val gtpa = getOrCreateGTPA(level.dimensionId)

    val existingJoint = gtpa.getJointById(jointId) ?: return

    val newPose0 = VSJointPose(
        newPos0,
        existingJoint.pose0.rot
    )
    val newJoint = VSFixedJoint(
        existingJoint.shipId0,
        newPose0,
        existingJoint.shipId1,
        existingJoint.pose1
    )
    val jointAndId = VSJointAndId(jointId, newJoint)

    gtpa.updateJoint(jointAndId)
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public void updateJoint(Level level, Vector3d newPos0, int jointId) {
    GameToPhysicsAdapter gtpa = ValkyrienSkiesMod.getOrCreateGTPA(
        VSGameUtilsKt.getDimensionId(level)
    );

    VSFixedJoint existingJoint = (VSFixedJoint) gtpa.getJointById(jointId);
    if (existingJoint == null) return;

    VSJointPose newPose0 = new VSJointPose(
            newPos0,
            existingJoint.getPose0().getRot()
    );
    VSFixedJoint newJoint = new VSFixedJoint(
            existingJoint.getShipId0(),
            newPose0,
            existingJoint.getShipId1(),
            existingJoint.getPose1(),
            existingJoint.getMaxForceTorque(),
            existingJoint.getCompliance()
    );
    VSJointAndId jointAndId = new VSJointAndId(jointId, newJoint);
    
    gtpa.updateJoint(jointAndId);
}
</code-block>
</tab>
</tabs>

### Removing the joint

To remove a joint, you simply give the GTPA a `VSJointId`.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
fun removeJoint(level: Level, jointId: VSJointId) {
    val gtpa = getOrCreateGTPA(level.dimensionId)
    gtpa.removeJoint(jointId)
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public void removeJoint(Level level, int jointId) {
    GameToPhysicsAdapter gtpa = ValkyrienSkiesMod.getOrCreateGTPA(
            VSGameUtilsKt.getDimensionId(level)
    );
    gtpa.removeJoint(jointId);
}
</code-block>
</tab>
</tabs>