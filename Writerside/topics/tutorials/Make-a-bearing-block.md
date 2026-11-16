# Make a bearing block

This tutorial will guide your through making a Bearing block from scratch. 
The finished block will attach to a ship above it, allows it to spin along only one axis, and will be fully compatible with schematics.

You will learn how to:
* Save NBT values in your Block Entity
* Use the onUse method in your Block
* Find a ship above the block
* Construct a new joint for use as a bearing
* Attach that joint to ships, and modify it
* Make the block copy/pastable in schematics

## Before you start

You should already know how to:
- Register a custom block (with its own `Block` extending class)
- Register a custom block entity (with its own `BlockEntity` extending class)

If you don't already know this, you should check out other tutorials first.
[Kaupenjoe](https://www.youtube.com/@ModdingByKaupenjoe) and [TurtyWurty](https://www.youtube.com/@TurtyWurty) are good sources for learning how to do these basic modding tasks.

## Step 1. Basic setup

You should be starting out with a block and its corresponding block entity.
We will be editing both of these in this tutorial, make sure you don't mix them up when using the code examples from here.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlock(properties: Properties) : Block(properties) {
    
}

// Not in the same file
class ExampleBlockEntity(type: BlockEntityType&lt;*&gt;, pos: BlockPos, blockState: BlockState) : BlockEntity(type, pos, blockState) {

}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlock extends Block {
    public ExampleBlock(Properties properties) {
        super(properties);
    }
}

// Not in the same file
public class ExampleBlockEntity extends BlockEntity {
    public ExampleBlockEntity(BlockEntityType&lt;?&gt; type, BlockPos pos, BlockState blockState) {
        super(type, pos, blockState);
    }
}
</code-block>
</tab>
</tabs>

Next we want to add a nbt variable `assembled` which we store on block unload to know if we already have an attached ship.
We add this to the block entity:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlockEntity(type: BlockEntityType&gt;*&lt;, pos: BlockPos, blockState: BlockState) :
    BlockEntity(type, pos, blockState) {

    var assembled = false

    override fun saveAdditional(tag: CompoundTag) {
        super.saveAdditional(tag)
        tag.putBoolean("assembled", assembled)
    }

    override fun load(tag: CompoundTag) {
        super.load(tag)
        assembled = tag.getBoolean("assembled")
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlockEntity extends BlockEntity {
    public boolean assembled = false;

    public ExampleBlockEntity(BlockEntityType&lt;?&gt; type, BlockPos pos, BlockState blockState) {
        super(type, pos, blockState);
    }

    @Override
    protected void saveAdditional(@NotNull CompoundTag tag) {
        super.saveAdditional(tag);
        tag.putBoolean("assembled", assembled);
    }

    @Override
    public void load(@NotNull CompoundTag tag) {
        super.load(tag);
        assembled = tag.getBoolean("assembled");
    }
}
</code-block>
</tab>
</tabs>

The `saveAdditional` and `load` make sure that the `assembled` value persists between world/chunk reloads.

Now we hook into the player interact method in the _block_ class:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlock(properties: Properties) : Block(properties) {
    override fun use(
        state: BlockState,
        level: Level,
        pos: BlockPos,
        player: Player,
        hand: InteractionHand,
        hit: BlockHitResult
    ): InteractionResult {
        val be = level.getBlockEntity(pos)
        if (be !is ExampleBlockEntity) return InteractionResult.FAIL

        // Do some logic
        
        be.assembled = true
        return InteractionResult.SUCCESS
    }
}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlock extends Block {
    public ExampleBlock(Properties properties) {
        super(properties);
    }

    @Override
    public @NotNull InteractionResult use(
        @NotNull BlockState state, 
        @NotNull Level level, 
        @NotNull BlockPos pos, 
        @NotNull Player player, 
        @NotNull InteractionHand hand, 
        @NotNull BlockHitResult hit
    ) {
        if (!(level.getBlockEntity(pos) instanceof ExampleBlockEntity be)) return InteractionResult.FAIL;

        // Do some logic

        be.assembled = true;
        return InteractionResult.SUCCESS;
    }
}
</code-block>
</tab>
</tabs>

## Step 2. Locating existing ships

Our bearing will first look for an existing ship positioned above it, and try to attach. 
Only if that fails will it assemble its own ship.

To find a ship, we just use a raycast upwards from the bearing.

> If your bearing can be rotated sideways, your raycast will need to follow that direction. 
> Our example bearing is only ever placed upwards, for simplicity.
>  
{style="note"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlock(properties: Properties) : Block(properties) {
    override fun use(
        state: BlockState,
        level: Level,
        pos: BlockPos,
        player: Player,
        hand: InteractionHand,
        hit: BlockHitResult
    ): InteractionResult {
        val be = level.getBlockEntity(pos)
        if (be !is ExampleBlockEntity) return InteractionResult.FAIL

        // Only continue if on the server
        val level = level as? ServerLevel ?: return InteractionResult.PASS

        // Change this if your bearing can rotate its blockstate
        val direction = Direction.UP

        // The position above us to attach a ship to
        val attachPoint = pos.relative(direction)

        // The ship the bearing is on, if any
        val currentShip = level.getShipManagingPos(pos)

        // The center of the bearing position, in a JOML vector
        val centerPos = pos.center.toJOML()

        // The direction our bearing is facing, in a JOML vector
        // Facing up it would be (0.0, 1.0, 0.0)
        val axis = direction.normal.toJOMLD()

        // We calculate these before transforming out of the shipyard because our "up"
        // may not be the same as worldspace "up"
        var startPos = Vector3d(centerPos).fma(0.5, axis)
        var endPos = Vector3d(centerPos).fma(1.5, axis)

        // Transform the positions to world for the raycast
        // If the ship is null, leave the positions alone
        startPos = (currentShip?.transform?.shipToWorld?.transformPosition(startPos) ?: startPos)
        endPos = (currentShip?.transform?.shipToWorld?.transformPosition(endPos) ?: endPos)

        // Do the raycast
        val otherPos = level.clipIncludeShips(
            ClipContext(
                startPos.toMinecraft(),
                endPos.toMinecraft(),
                ClipContext.Block.COLLIDER,
                ClipContext.Fluid.NONE,
                null
            ), false, currentShip?.id
        )

        // .takeUnless means that if the ship we raycast is the same ship as we are, just make otherShip null
        val otherShip = level.getLoadedShipManagingPos(otherPos.blockPos)?.takeUnless { it.id == currentShip?.id }

        // TODO: More logic

        be.assembled = true
        return InteractionResult.SUCCESS
    }
}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlock extends Block {
    public ExampleBlock(Properties properties) {
        super(properties);
    }

    @Override
    public @NotNull InteractionResult use(@NotNull BlockState state, @NotNull Level level, @NotNull BlockPos pos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult hit) {
        if (!(level.getBlockEntity(pos) instanceof ExampleBlockEntity be)) return InteractionResult.FAIL;

        // Only continue if on the server
        if (!(level instanceof ServerLevel serverLevel)) {
            return InteractionResult.PASS;
        }

        // Change this if your bearing can rotate its blockstate
        Direction direction = Direction.UP;

        // The position above us to attach a ship to
        BlockPos attachPoint = pos.relative(direction);

        // The ship the bearing is on, if any
        Ship currentShip = VSGameUtilsKt.getShipManagingPos(level, pos);

        // The center of the bearing position, in a JOML vector
        Vector3dc centerPos = VectorConversionsMCKt.toJOML(pos.getCenter());

        // The direction our bearing is facing, in a JOML vector
        // Facing up it would be (0.0, 1.0, 0.0)
        Vector3dc axis = VectorConversionsMCKt.toJOMLD(direction.getNormal());

        // We calculate these before transforming out of the shipyard because our "up"
        // may not be the same as worldspace "up"
        Vector3d startPos = new Vector3d(centerPos).fma(0.5, axis);
        Vector3d endPos = new Vector3d(centerPos).fma(1.5, axis);

        // Transform the positions to world for the raycast
        // If the ship is null, leave the positions alone
        if (currentShip != null) {
            startPos = currentShip.getTransform().getShipToWorld().transformPosition(startPos);
            endPos = currentShip.getTransform().getShipToWorld().transformPosition(endPos);
        }

        // Do the raycast
        BlockHitResult otherPos = RaycastUtilsKt.clipIncludeShips(level,
                new ClipContext(
                        VectorConversionsMCKt.toMinecraft(startPos),
                        VectorConversionsMCKt.toMinecraft(endPos),
                        ClipContext.Block.COLLIDER,
                        ClipContext.Fluid.NONE,
                        null
                ), false, (currentShip != null) ? currentShip.getId() : null
        );

        Ship otherShip = VSGameUtilsKt.getLoadedShipManagingPos(level, otherPos.getBlockPos());
        // if the ship we raycast is the same ship as we are, just make otherShip null
        if (otherShip != null && currentShip != null && otherShip.getId() == currentShip.getId()) {
            otherShip = null;
        }

        // TODO: More logic

        be.assembled = true;
        return InteractionResult.SUCCESS;
    }
}
</code-block>
</tab>
</tabs>

## Step 3. Making the joint

We now need to make a joint instance to attach our two ships. 
We will be using a VSRevoluteJoint, as it allows free rotation but only on a single axis, and has locked position.

> In this step we add the utility function getRotation, 
> make sure you don't miss it when copying code
> 
{style="warning"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlock(properties: Properties) : Block(properties) {
    @OptIn(GameTickOnly::class)
    override fun use(
        state: BlockState,
        level: Level,
        pos: BlockPos,
        player: Player,
        hand: InteractionHand,
        hit: BlockHitResult
    ): InteractionResult {
        val be = level.getBlockEntity(pos)
        if (be !is ExampleBlockEntity) return InteractionResult.FAIL

        // Only continue if on the server
        val level = level as? ServerLevel ?: return InteractionResult.PASS

        // Change this if your bearing can rotate its blockstate
        val direction = Direction.UP

        // The position above us to attach a ship to
        val attachPoint = pos.relative(direction)

        // The ship the bearing is on, if any
        val currentShip = level.getShipManagingPos(pos)

        // The center of the bearing position, in a JOML vector
        val centerPos = pos.center.toJOML()

        // The direction our bearing is facing, in a JOML vector
        // Facing up it would be (0.0, 1.0, 0.0)
        val axis = direction.normal.toJOMLD()

        // We calculate these before transforming out of the shipyard because our "up"
        // may not be the same as worldspace "up"
        var startPos = Vector3d(centerPos).fma(0.5, axis)
        var endPos = Vector3d(centerPos).fma(1.5, axis)

        // Transform the positions to world for the raycast
        // If the ship is null, leave the positions alone
        startPos = (currentShip?.transform?.shipToWorld?.transformPosition(startPos) ?: startPos)
        endPos = (currentShip?.transform?.shipToWorld?.transformPosition(endPos) ?: endPos)

        // Do the raycast
        val otherPos = level.clipIncludeShips(
            ClipContext(
                startPos.toMinecraft(),
                endPos.toMinecraft(),
                ClipContext.Block.COLLIDER,
                ClipContext.Fluid.NONE,
                null
            ), false, currentShip?.id
        )

        // .takeUnless means that if the ship we raycast is the same ship as we are, just make otherShip null
        val otherShip = level.getLoadedShipManagingPos(otherPos.blockPos)?.takeUnless { it.id == currentShip?.id }

        // Safely get a nullable ship ID
        val currentId = currentShip?.id
        val otherId = otherShip?.id

        // Get hinge rotation quaternion using some magic math
        val hingeRot = ExampleBlock.getRotation(direction)

        // The position of the joint on our main ship (which may also be the world)
        val pose0 = VSJointPose(
            centerPos,
            hingeRot
        )

        // The position of the joint on our other ship.
        // The joint will try to keep the two positions in the same place,
        // and the rotations the same
        val pose1 = VSJointPose(
            otherPos.blockPos.center.toJOML(),
            hingeRot
        )

        val joint = VSRevoluteJoint(
            currentId,
            pose0,
            otherId,
            pose1,
            null,
            // Compliance controls how "loose" your joint is. It should generally be a very small value
            1e-20,
            // Every parameter below this line only works on certain physics backends.
            // It's not recommended you use them.
            null,
            null,
            null,
            null,
            null
        )
        
        // TODO: More logic

        be.assembled = true
        return InteractionResult.SUCCESS
    }

    // Taken and converted to Java in pieces from TestHingeBlock
    fun getRotation(direction: Direction): Quaterniondc {
        // The rotation we apply to different face values. The code below is set up to create Y-hinges by
        // default, and [rotationQuaternion] converts them to other types of hinges
        val rotationQuaternion = when (direction) {
            DOWN ->  Quaterniond(AxisAngle4d(Math.PI, Vector3d(1.0, 0.0, 0.0)))
            NORTH -> Quaterniond(AxisAngle4d(Math.PI, Vector3d(0.0, 1.0, 0.0))).mul(Quaterniond(AxisAngle4d(Math.PI / 2.0, Vector3d(1.0, 0.0, 0.0)))).normalize()
            EAST ->  Quaterniond(AxisAngle4d(0.5 * Math.PI, Vector3d(0.0, 1.0, 0.0))).mul(Quaterniond(AxisAngle4d(Math.PI / 2.0, Vector3d(1.0, 0.0, 0.0)))).normalize()
            SOUTH -> Quaterniond(AxisAngle4d(Math.PI / 2.0, Vector3d(1.0, 0.0, 0.0))).normalize()
            WEST ->  Quaterniond(AxisAngle4d(1.5 * Math.PI, Vector3d(0.0, 1.0, 0.0))).mul(Quaterniond(AxisAngle4d(Math.PI / 2.0, Vector3d(1.0, 0.0, 0.0)))).normalize()
            UP -> Quaterniond()
        }

        // Hinge constraints will attempt to align the X-axes of both bodies, so to align the Y axis we
        // apply this rotation to the X-axis
        return rotationQuaternion.mul(Quaterniond(AxisAngle4d(Math.toRadians(90.0), 0.0, 0.0, 1.0)), Quaterniond())
            .normalize()
    }
}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlock extends Block {
    public ExampleBlock(Properties properties) {
        super(properties);
    }

    @Override
    public @NotNull InteractionResult use(@NotNull BlockState state, @NotNull Level level, @NotNull BlockPos pos, @NotNull Player player, @NotNull InteractionHand hand, @NotNull BlockHitResult hit) {
        if (!(level.getBlockEntity(pos) instanceof ExampleBlockEntity be)) return InteractionResult.FAIL;

        // Only continue if on the server
        if (!(level instanceof ServerLevel serverLevel)) {
            return InteractionResult.PASS;
        }

        // Change this if your bearing can rotate its blockstate
        Direction direction = Direction.UP;

        // The position above us to attach a ship to
        BlockPos attachPoint = pos.relative(direction);

        // The ship the bearing is on, if any
        Ship currentShip = VSGameUtilsKt.getShipManagingPos(level, pos);

        // The center of the bearing position, in a JOML vector
        Vector3dc centerPos = VectorConversionsMCKt.toJOML(pos.getCenter());

        // The direction our bearing is facing, in a JOML vector
        // Facing up it would be (0.0, 1.0, 0.0)
        Vector3dc axis = VectorConversionsMCKt.toJOMLD(direction.getNormal());

        // We calculate these before transforming out of the shipyard because our "up"
        // may not be the same as worldspace "up"
        Vector3d startPos = new Vector3d(centerPos).fma(0.5, axis);
        Vector3d endPos = new Vector3d(centerPos).fma(1.5, axis);

        // Transform the positions to world for the raycast
        // If the ship is null, leave the positions alone
        if (currentShip != null) {
            startPos = currentShip.getTransform().getShipToWorld().transformPosition(startPos);
            endPos = currentShip.getTransform().getShipToWorld().transformPosition(endPos);
        }

        // Do the raycast
        BlockHitResult otherPos = RaycastUtilsKt.clipIncludeShips(level,
                new ClipContext(
                        VectorConversionsMCKt.toMinecraft(startPos),
                        VectorConversionsMCKt.toMinecraft(endPos),
                        ClipContext.Block.COLLIDER,
                        ClipContext.Fluid.NONE,
                        null
                ), false, (currentShip != null) ? currentShip.getId() : null
        );

        Ship otherShip = VSGameUtilsKt.getLoadedShipManagingPos(level, otherPos.getBlockPos());

        // Safely get a nullable ship ID
        Long currentId = (currentShip != null) ? currentShip.getId() : null;
        Long otherId = (otherShip != null) ? otherShip.getId() : null;

        // If the ship we raycast is the same ship as we are, just make otherShip null
        if (Objects.equals(currentId, otherId)) {
            otherShip = null;
        }

        // Get hinge rotation quaternion using some magic math
        Quaterniondc hingeRot = getRotation(direction);

        // The position of the joint on our main ship (which may also be the world)
        VSJointPose pose0 = new VSJointPose(
                centerPos,
                hingeRot
        );

        // The position of the joint on our other ship.
        // The joint will try to keep the two positions in the same place,
        // and the rotations the same
        VSJointPose pose1 = new VSJointPose(
                VectorConversionsMCKt.toJOML(otherPos.getBlockPos().getCenter()),
                hingeRot
        );


        VSRevoluteJoint joint = new VSRevoluteJoint(
                currentId,
                pose0,
                otherId,
                pose1,
                null,
                // Compliance controls how "loose" your joint is. It should generally be a very small value
                1e-20,
                // Every parameter below this line only works on certain physics backends.
                // It's not recommended you use them.
                null,
                null,
                null,
                null,
                null
        );

        // TODO: More logic

        be.assembled = true;
        return InteractionResult.SUCCESS;
    }

    // Taken and converted to Java in pieces from TestHingeBlock
    public static Quaterniondc getRotation(Direction direction) {
        // The rotation we apply to different face values. The code below is set up to create Y-hinges by
        // default, and [rotationQuaternion] converts them to other types of hinges
        Quaterniondc rotationQuaternion = switch (direction) {
            case DOWN -> new Quaterniond(new AxisAngle4d(Math.PI, new Vector3d(1.0, 0.0, 0.0)));
            case NORTH ->
                    new Quaterniond(new AxisAngle4d(Math.PI, new Vector3d(0.0, 1.0, 0.0))).mul(new Quaterniond(new AxisAngle4d(Math.PI / 2.0, new Vector3d(1.0, 0.0, 0.0)))).normalize();
            case EAST ->
                    new Quaterniond(new AxisAngle4d(0.5 * Math.PI, new Vector3d(0.0, 1.0, 0.0))).mul(new Quaterniond(new AxisAngle4d(Math.PI / 2.0, new Vector3d(1.0, 0.0, 0.0)))).normalize();
            case SOUTH -> new Quaterniond(new AxisAngle4d(Math.PI / 2.0, new Vector3d(1.0, 0.0, 0.0))).normalize();
            case WEST ->
                    new Quaterniond(new AxisAngle4d(1.5 * Math.PI, new Vector3d(0.0, 1.0, 0.0))).mul(new Quaterniond(new AxisAngle4d(Math.PI / 2.0, new Vector3d(1.0, 0.0, 0.0)))).normalize();
            case UP -> new Quaterniond();
        };

        // Hinge constraints will attempt to align the X-axes of both bodies, so to align the Y axis we
        // apply this rotation to the X-axis
        return rotationQuaternion.mul(new Quaterniond(new AxisAngle4d(Math.toRadians(90.0), 0.0, 0.0, 1.0)), new Quaterniond()).normalize();
    }
}
</code-block>
</tab>
</tabs>

## Step 4. Adding the joint

After constructing the joint instance, we still need to add it to the world.

Replace the `// TODO: More logic` in the previous code with this new logic:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val gtpa = getOrCreateGTPA(level.dimensionId)
gtpa.addJoint(joint, 0) { jointId: Int -> }
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
GameToPhysicsAdapter gtpa = ValkyrienSkiesMod.getOrCreateGTPA(VSGameUtilsKt.getDimensionId(level));
gtpa.addJoint(joint, 0, (jointId) -> {});
</code-block>
</tab>
</tabs>

However this will leave us with a joint that we can't remove or modify, since we need the joint ID to do so.

To fix this, let's go back to our _block entity_ and add a new NBT value to store our joint id.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlockEntity(type: BlockEntityType&lt;*&gt;, pos: BlockPos, blockState: BlockState) :
    BlockEntity(type, pos, blockState) {

    var assembled = false
    var jointId = -1

    override fun saveAdditional(tag: CompoundTag) {
        super.saveAdditional(tag)
        tag.putBoolean("assembled", assembled)
        tag.putInt("jointId", jointId)
    }

    override fun load(tag: CompoundTag) {
        super.load(tag)
        assembled = tag.getBoolean("assembled")
        jointId = tag.getInt("jointId")
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlockEntity extends BlockEntity {
    public boolean assembled = false;
    public int jointId = -1;

    public ExampleBlockEntity(BlockEntityType&lt;?&gt; type, BlockPos pos, BlockState blockState) {
        super(type, pos, blockState);
    }

    @Override
    protected void saveAdditional(@NotNull CompoundTag tag) {
        super.saveAdditional(tag);
        tag.putBoolean("assembled", assembled);
        tag.putInt("jointId", jointId);
    }

    @Override
    public void load(@NotNull CompoundTag tag) {
        super.load(tag);
        assembled = tag.getBoolean("assembled");
        jointId = tag.getInt("jointId");
    }
}

</code-block>
</tab>
</tabs>

Now lets update it back in our block class:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val gtpa = getOrCreateGTPA(level.dimensionId)
gtpa.addJoint(joint, 0) { jointId: Int -> 
    be.jointId = jointId
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
GameToPhysicsAdapter gtpa = ValkyrienSkiesMod.getOrCreateGTPA(VSGameUtilsKt.getDimensionId(level));
gtpa.addJoint(joint, 0, (jointId) -> {
    be.jointId = jointId;
});
</code-block>
</tab>
</tabs>

If you tested it at this point, you would discover that although our `jointId` persists between world reloads,
our joint itself does not!

There's an easy fix for this. Right after we create the joint, we just need to mark it as "serialized".

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val joint = // ...
joint.serialized()
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
VSRevoluteJoint joint = // ...
joint.serialized();
</code-block>
</tab>
</tabs>

## Step 5. Homework

Now you've (hopefully) got a working joint, that attaches to ships and keeps its data after world reload.

Just to be annoying, and to make sure you aren't copy-pasting this code blindly, I'm going to give you an exercise to do yourself.

See if you can, on your own, make it so that if `onUse` is called while the bearing is already `assembled=true`
it removes the existing joint with the `jointId`.

Some hints:
- You already store `assembled` and `jointId` in the `be` (block entity) variable, ready to be accessed
- You can remove a joint with a given id using the `gtpa`, similar to how you add the joint

## Step 6. Making it work in schematics

You may notice that when you paste in schematics with this bearing, 
the joint is duplicated to the new ship but the block no longer has the right jointId.

Luckily there is a solution designed exactly for this.

We just need to implement `ICopyableBlock` on our Block class, and use its joint id migration.

For more details on `ICopyableBlock`, see [this guide](How-to-use-ICopyableBlock.md)

Here is what it should roughly look like:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class ExampleBlock(properties: Properties) : Block(properties), ICopyableBlock {
    
    // Everything else

    override fun onCopy(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        be: BlockEntity?,
        shipsBeingCopied: List&lt;ServerShip&gt;,
        centerPositions: Map&lt;Long, Vector3dc&gt;
    ): CompoundTag? {
        // null = leave it up to the block entity saveAdditional to make the CompoundTag
        return null
    }

    override fun onPaste(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        oldShipIdToNewId: Map&lt;Long, Long&gt;,
        centerPositions: Map&lt;Long, Pair&lt;Vector3dc, Vector3dc&gt;&gt;,
        oldJointIdToNewId: Map&lt;Int, Int&gt;,
        tag: CompoundTag?
    ): CompoundTag? {
        if (tag == null) return null
        val oldId = tag.getInt("jointId")

        // If the joint id isn't found in the migration map, use an id of -1 to "reset" our block
        // (e.g. wipe its joint data). This should never occur if our joint was copied properly though.
        val newId = oldJointIdToNewId.getOrDefault(oldId, -1)

        // Overwrite the NBT
        tag.putInt("jointId", newId)
        return tag
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class ExampleBlock extends Block implements ICopyableBlock {
    
    // Everything else

    @Override
    public @Nullable CompoundTag onCopy(
            @NotNull ServerLevel serverLevel,
            @NotNull BlockPos pos,
            @NotNull BlockState state,
            @Nullable BlockEntity be,
            @NotNull List&lt;? extends ServerShip&gt; shipsBeingCopied,
            @NotNull Map&lt;Long, ? extends Vector3dc&gt; centerPositions
    ) {
        // null = leave it up to the block entity saveAdditional to make the CompoundTag
        return null;
    }

    @Override
    public @Nullable CompoundTag onPaste(
            ServerLevel level,
            BlockPos pos,
            BlockState state,
            Map&lt;Long, Long&gt; oldShipIdToNewId,
            Map&lt;Long, Pair&lt;Vector3dc, Vector3dc&gt;&gt; centerPositions,
            Map&lt;Integer, Integer&gt; oldJointIdToNewId,
            @Nullable CompoundTag tag
    ) {
        if (tag == null) return null;
        int oldId = tag.getInt("jointId");

        // If the joint id isn't found in the migration map, use an id of -1 to "reset" our block
        // (e.g. wipe its joint data). This should never occur if our joint was copied properly though.
        int newId = oldJointIdToNewId.getOrDefault(oldId, -1);

        // Overwrite the NBT
        tag.putInt("jointId", newId);
        return tag;
    }
}
</code-block>
</tab>
</tabs>

## Step 7. Next steps

There's still a lot not covered in this tutorial. 

You may, for example, want your bearing to assemble a new ship instead of attaching to an existing one. For that you can check out [](How-to-assemble-a-ship.md).

Or you may want to use a different joint type, or have a different rotation, or something else entirely.

Unfortunately, although we are expanding these docs all the time, we can't provide a tutorial for everything.

Some of your desired features will need to be figured out with your own experimentation, although the source code of other addons can always help.

But hopefully this tutorial gave you a good overview of the standard steps your bearing needs.