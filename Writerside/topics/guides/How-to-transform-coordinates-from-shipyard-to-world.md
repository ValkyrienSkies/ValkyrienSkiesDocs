# How to transform position from shipyard to world

Learn how to convert a position in the [shipyard](The-Shipyard.md) where blocks on a ship are stored, to world 
coordinates, where those blocks actually appear.

## Before you start

Make sure that:
- You have added the Valkyrien Skies API to your dev environment.
- You are using VS 2.5 or later. 

> Positions in the shipyard always have an x and z value in the millions.
> 
{style="note"}

## Convert a shipyard position using Level

Convert a shipyard position the easiest way, using the `Level` class.

### Convert a position represented by a Minecraft `Vec3`

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

### Converted a position represented by a JOML `Vector3d`

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

## Convert a shipyard position using Ship

Convert a shipyard position using the ship that contains it. First, you'll have to get the Ship which manages that 
position.

### Convert a position represented by a Minecraft `Vec3` {id="convert-a-position-represented-by-a-minecraft-vec3_1"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val ship: Ship = ...
val shipPos: Vec3 = ...
val worldPos = ValkyrienSkies.positionToWorld(ship, shipPos)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Ship ship = ...
Vec3 shipPos = ...
Vec3 worldPos = ValkyrienSkies.positionToWorld(ship, shipPos);
</code-block>
</tab>
</tabs>

### Converted a position represented by a JOML `Vector3d` {id="converted-a-position-represented-by-a-joml-vector3d_1"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val ship: Ship = ...
val shipPos: Vector3dc = ...
val worldPos = ValkyrienSkies.positionToWorld(ship, shipPos, Vector3d())
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
Ship ship = ...
Vector3dc shipPos = ...
Vector3d worldPos = ValkyrienSkies.positionToWorld(ship, shipPos, new Vector3d());
</code-block>
</tab>
</tabs>
