# How to transform a position from shipyard to world

Learn how to transform a position in the [shipyard](The-Shipyard.md) where blocks on a [ship](Ships.md) are stored, to 
world coordinates, where those blocks actually appear.

> Positions in the shipyard always have an x and z value in the millions.
>
{style="note"}

## Before you start

Make sure that:
- You have [added the Valkyrien Skies API to your dev environment](How-to-add-Valkyrien-Skies-to-your-dev-environment.md).

> If you try to transform a position that's not [claimed](The-Shipyard.md#chunk-claims) by a ship, the original 
> position will be returned unmodified.
> 
{style="warning"}

## Transform a shipyard position using Level

### Transform a position represented by a Minecraft `Vec3`

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val level: Level = ...
val shipPos: Vec3 = ...
val worldPos = level.positionToWorld(shipPos)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Level level = ...
Vec3 shipPos = ...
Vec3 worldPos = ValkyrienSkies.positionToWorld(level, shipPos);
</code-block>
</tab>
</tabs>

### Transform a position represented by a JOML `Vector3d`

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val level: Level = ...
val shipPos: Vector3dc = ...
val worldPos = level.positionToWorld(shipPos, Vector3d())
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Level level = ...
Vector3dc shipPos = ...
Vector3d worldPos = ValkyrienSkies.positionToWorld(level, shipPos, new Vector3d());
</code-block>
</tab>
</tabs>

## Transform a shipyard position using Ship

> To transform a shipyard position using a ship, you need to 
> [get the ship managing that position](How-to-get-a-ship.md#using-a-shipyard-position).

### Transform a position represented by a Minecraft `Vec3` {id="convert-a-position-represented-by-a-minecraft-vec3_1"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val shipPos: Vec3 = ...
val ship: Ship = ...
val worldPos: Vector3d = ship.positionToWorld(shipPos)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Vec3 shipPos = ...
Ship ship = ...
Vec3 worldPos = ValkyrienSkies.positionToWorld(ship, shipPos);
</code-block>
</tab>
</tabs>

### Transformed a position represented by a JOML `Vector3d` {id="converted-a-position-represented-by-a-joml-vector3d_1"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val shipPos: Vector3dc = ...
val ship: Ship = ...
val worldPos: Vector3d = ship.positionToWorld(shipPos, Vector3d())
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Vector3dc shipPos = ...
Ship ship = ...
Vector3d worldPos = ValkyrienSkies.positionToWorld(ship, shipPos, new Vector3d());
</code-block>
</tab>
</tabs>
