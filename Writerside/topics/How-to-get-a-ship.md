# How to get a ship

Learn how to get an instance of `Ship` using a shipyard position, world position, or ship id.

## Before you start

Make sure that:
- You have [added Valkyrien Skies to your dev environment](How-to-add-Valkyrien-Skies-to-your-dev-environment.md).

## Using a shipyard position 

Get the ship whose unique [chunk claim](The-Shipyard.md#chunk-claims) in the shipyard contains a specific block or
chunk, if it exists.

> A ship is said to be **managing** chunks and blocks which are contained within its chunk claim.
>
{style="note"}

### Get the ship managing a block position

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.api.getShipManagingBlock

val level: Level = ...
val ship: Ship? = level.getShipManagingBlock(x, y, z)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
import org.valkyrienskies.mod.api.ValkyrienSkies

Level level = ...
@Nullable Ship ship = ValkyrienSkies.getShipManagingBlock(level, x, y, z);
</code-block>
</tab>
</tabs>

### Get the ship managing a chunk position 

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.api.getShipManagingChunk

val level: Level = ...
val ship: Ship? = level.getShipManagingChunk(chunkX, chunkZ)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
import org.valkyrienskies.mod.api.ValkyrienSkies

Level level = ...
@Nullable Ship ship = ValkyrienSkies.getShipManagingChunk(level, chunkX, chunkZ);
</code-block>
</tab>
</tabs>

## Using a world position

### Get ships containing a position

Get all the ships with bounding boxes that contain a given position. It's possible for multiple ships to contain
the same position, so these methods return an `Iterable<Ship>`.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val level: Level = ...
val ships: Iterable&lt;Ship&gt; = level.getShipsIntersecting(x, y, z)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Level level = ...
Iterable&lt;Ships&gt; ships = ValkyrienSkies.getShipsIntersecting(level, x, y, z);
</code-block>
</tab>
</tabs>

### Get the ships intersecting a bounding box

Get all the ships with bounding boxes that intersect a given bounding box.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val level: Level = ...
val aabb: AABB = ...
val ships: Iterable&lt;Ship&gt; = level.getShipsIntersecting(aabb)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Level level = ...
AABB aabb = ...
Iterable&lt;Ships&gt; ships = ValkyrienSkies.getShipsIntersecting(level, aabb);
</code-block>
</tab>
</tabs>

## Using the ship's ID

Get the ship with the specified ID, if it exists.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val level: Level = ...
val id: ShipId = ...
val ship: Ship? = level.getShipById(id)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Level level = ...
long id = ...
@Nullable Ship ship = ValkyrienSkies.getShipById(level, id);
</code-block>
</tab>
</tabs>