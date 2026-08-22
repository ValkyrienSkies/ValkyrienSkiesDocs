# How to use ICopyableBlock

Learn how to use the ICopyableBlock interface to improve compatibility with assembly, schematics, and splitting.

## Introduction

ICopyableBlock is an interface which can be implemented by any custom class that extends Block.

The methods you implement will be run by Valkyrien Skies itself (as well as addons like VMod) to improve compatibility
with transitions such as schematic copy/pasting or ship splitting.

It can be used for a wide variety of blocks to improve compat. Examples include:
- Bearing block being pasted in schematics without losing its joint
- Cross-ship fluid pipe being split off into a second ship without losing its connection
- Control seat persisting its linked blocks after being pasted as a new ship

If your blocks are having issues with VMod schematics, you're in the right place.

## Steps

### Implementing the interface

The first step is to add the interface to your custom block class. 
You can also add it to another mods block using a mixin, but that won't be covered in this guide.

> Don't accidentally implement ICopyableBlock on your **block entity**, 
> it will not be called. It must be implemented on your Block class. 
>
{style="warning"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class MyCustomBlock(properties: Properties): Block(properties), ICopyableBlock {
    // ...
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyCustomBlock extends Block implements ICopyableBlock {
    public MyCustomBlock(Properties properties) {
        super(properties);
    }
    // ...
}
</code-block>
</tab>
</tabs>

Then, you must implement two functions, `onCopy` and `onPaste`. (Your IDE will often provide the method signature for you)

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class MyCustomBlock(properties: Properties): Block(properties), ICopyableBlock {
    override fun onCopy(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        be: BlockEntity?,
        shipsBeingCopied: List&lt;ServerShip&gt;,
        centerPositions: Map&lt;Long, Vector3dc&gt;
    ): CompoundTag? {
        return null
    }

    override fun onPaste(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        oldShipIdToNewId: Map&lt;Long, Long&gt;,
        centerPositions: Map&lt;Long, Pair&lt;Vector3dc, Vector3dc&gt;&gt;,
        tag: CompoundTag?
    ): CompoundTag? {
        return null
    }

}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyCustomBlock extends Block implements ICopyableBlock {
    public MyCustomBlock(Properties properties) {
        super(properties);
    }

    @Override
    public CompoundTag onCopy(
        ServerLevel serverLevel, 
        BlockPos blockPos, 
        BlockState blockState, 
        BlockEntity blockEntity, 
        List&lt;? extends ServerShip&gt; shipsBeingCopied, 
        Map&lt;Long, ? extends Vector3dc&gt; centerPositions
    ) {
        return null;
    }

    @Override
    public CompoundTag onPaste(
        ServerLevel serverLevel, 
        BlockPos blockPos, 
        BlockState blockState, 
        Map&lt;Long, Long&gt; oldShipIdToNewId, 
        Map&lt;Long, ? extends Pair&lt;? extends Vector3dc, ? extends Vector3dc&gt;&gt; centerPositions, 
        CompoundTag tag
    ) {
        return null;
    }
}
</code-block>
</tab>
</tabs>

### Understanding the interface

The Javadoc comments on these methods are rather technical, so it can be useful to see a high level overview of how they work.

When your block is copied, `onCopy` is called. The `CompoundTag` that `onCopy` returns will be saved. 

If `onCopy` returns `null`, then
the `CompoundTag` from the block entity `saveAdditional` is saved. Most blocks choose to return `null` from `onCopy` for simplicity, 
saving the block entity data.

When the block is pasted, `onPaste` is called. The tag you returned from `onCopy` or `saveAdditional` will be given to `onPaste`, as 
well as some additional information like a mapping of old ship ids (the ids from when your block was copied) to new ship ids 
(the ids of the ships your block is being pasted onto). 

Whatever compound tag you return from `onPaste` is loaded into the newly placed block, and is usually parsed by the 
block entity's `load` function. If `null` is returned from `onPaste`, it will load the unmodified compound tag into the block.

This means you can use `onPaste` as a form of "pre-processor" for the block entities data. You take in the data that was saved 
(for example the ship your bearing is jointed to), and map it to new data (like the new ship ids you have been pasted on).

This may sound complicated but once you wrap your head around it, it isn't that bad.

<code-block lang="mermaid">
graph TB
    subgraph Copy
        direction LR
        A[Ship is being copied]
        A --> B[onCopy]
        B -- Not null --> D
        B -- null --> C[Block Entity saveAdditional]
        C --> D
        D[Saved data]
    end
    subgraph Paste
        direction RL
        E[Saved data]
        E --> F[onPaste]
        F --> G[Block Entity load]
        E -- onPaste is null --> G
        G --> H[Ship has been pasted]
    end
    Copy --> Paste    

</code-block>

## Example

Below is an example of a simple block which stores a position on another ship,
and fully supports being duplicated to a new ship (e.g. when pasted in a schematic).

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class MyBearingBlock(properties: Properties) : Block(properties), ICopyableBlock {
    override fun onCopy(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        be: BlockEntity?,
        shipsBeingCopied: List&lt;ServerShip&gt;,
        centerPositions: Map&lt;Long, Vector3dc&gt;
    ): CompoundTag? {
        return null
    }

    override fun onPaste(
        level: ServerLevel,
        pos: BlockPos,
        state: BlockState,
        oldShipIdToNewId: Map&lt;Long, Long&gt;,
        centerPositions: Map&lt;Long, Pair&lt;Vector3dc, Vector3dc&gt;&gt;,
        tag: CompoundTag?
    ): CompoundTag? {
        tag ?: return null
        if (!tag.contains("x")) return null

        val x = tag.getDouble("x")
        val y = tag.getDouble("y")
        val z = tag.getDouble("z")
        val oldPosition = Vector3d(x, y, z)

        val oldShip = tag.getLong("connectedShip")

        // null if our old ship wasn't copied
        val centerMigrate: Pair&lt;Vector3dc, Vector3dc&gt; = centerPositions[oldShip] ?: return null

        // offset between old position to old center position
        val offset = oldPosition.sub(centerMigrate.first, Vector3d())

        val newPosition = centerMigrate.second.add(offset, Vector3d())

        // Write our new position back into the tag
        tag.putDouble("x", newPosition.x())
        tag.putDouble("y", newPosition.y())
        tag.putDouble("z", newPosition.z())

        return tag
    }

}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyBearingBlock extends Block implements ICopyableBlock {
    public MyBearingBlock(Properties properties) {
        super(properties);
    }

    @Override
    public CompoundTag onCopy(
            ServerLevel serverLevel,
            BlockPos blockPos,
            BlockState blockState,
            BlockEntity blockEntity,
            List&lt;? extends ServerShip&gt; shipsBeingCopied,
            Map&lt;Long, ? extends Vector3dc&gt; centerPositions
    ) {
        // Let the block entity save our data
        return null;
    }

    @Override
    public CompoundTag onPaste(
            ServerLevel serverLevel,
            BlockPos blockPos,
            BlockState blockState,
            Map&lt;Long, Long&gt; oldShipIdToNewId,
            Map&lt;Long, ? extends Pair&lt;? extends Vector3dc, ? extends Vector3dc&gt;&gt; centerPositions,
            CompoundTag tag
    ) {
        if (tag == null) return null;
        if (!tag.contains("x")) return null;

        double x = tag.getDouble("x");
        double y = tag.getDouble("y");
        double z = tag.getDouble("z");
        Vector3dc oldPosition = new Vector3d(x, y, z);

        long oldShip = tag.getLong("connectedShip");
        Pair&lt;? extends Vector3dc, ? extends Vector3dc&gt; centerMigrate = centerPositions.get(oldShip);

        // Our old ship wasn't copied
        if (centerMigrate == null) return null;

        // offset between old position to old center position
        Vector3dc offset = oldPosition.sub(centerMigrate.getFirst(), new Vector3d());

        Vector3dc newPosition = centerMigrate.getSecond().add(offset, new Vector3d());

        // Write our new position back into the tag
        tag.putDouble("x", newPosition.x());
        tag.putDouble("y", newPosition.y());
        tag.putDouble("z", newPosition.z());

        return tag;
    }
}
</code-block>
</tab>
</tabs>

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class MyBearingBlockEntity(type: BlockEntityType&lt;*&gt;, pos: BlockPos, blockState: BlockState) :
    BlockEntity(type, pos, blockState) {

    // Assume these are set from somewhere not shown
    var connectedPosition: BlockPos? = null
    var connectedShip: Long = -1

    override fun saveAdditional(tag: CompoundTag) {
        tag.putLong("connectedShip", connectedShip)

        connectedPosition?.let { pos ->
            // Using the center prevents precision issues
            // from when we migrate the position to a new ship
            tag.putDouble("x", pos.center.x)
            tag.putDouble("y", pos.center.y)
            tag.putDouble("z", pos.center.z)
        }
    }

    override fun load(tag: CompoundTag) {
        connectedShip = tag.getLong("connectedShip")

        if (!tag.contains("x")) return
        val x = tag.getDouble("x")
        val y = tag.getDouble("y")
        val z = tag.getDouble("z")
        connectedPosition = BlockPos.containing(x, y, z)
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyBearingBlockEntity extends BlockEntity {

    // Assume these are set from somewhere not shown
    public BlockPos connectedPosition = null;
    public long connectedShip = -1;

    public MyBearingBlockEntity(BlockEntityType&lt;?&gt; type, BlockPos pos, BlockState blockState) {
        super(type, pos, blockState);
    }

    @Override
    protected void saveAdditional(CompoundTag tag) {
        tag.putLong("connectedShip", connectedShip);

        if (connectedPosition == null) return;
        // Using the center prevents precision issues
        // from when we migrate the position to a new ship
        tag.putDouble("x", connectedPosition.getCenter().x);
        tag.putDouble("y", connectedPosition.getCenter().y);
        tag.putDouble("z", connectedPosition.getCenter().z);
    }

    @Override
    public void load(CompoundTag tag) {
        connectedShip = tag.getLong("connectedShip");

        if (!tag.contains("x")) return;
        double x = tag.getDouble("x");
        double y = tag.getDouble("y");
        double z = tag.getDouble("z");
        connectedPosition = BlockPos.containing(x, y, z);
    }
}
</code-block>
</tab>
</tabs>