# Ship Chunk Claims
Every ship has a chunk claim.
This chunk claim specifies what section of the shipyard the ship "owns".
Currently, chunk claims are hardcoded to 256 by 256 chunks, so ships can't be bigger than that.

## `chunkClaim` field in `Ship`
The function
```kotlin
fun getTotalVoxelRegion(yRange: LevelYRange, destination: AABBi = AABBi()): AABBi
```
in `ChunkClaim` can be used to get the AABB of the chunk claim (= Shipyard area)
It takes an argument of the type `LevelYRange` which can be easily created from a level:

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
new LevelYRange(level.getMinBuildHeight(), level.getMaxBuildHeight());
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
LevelYRange(level.minBuildHeight, level.maxBuildHeight)
```

</tab>
</tabs>

This creates a Y-Range using the build height of the level given.

## `chunkClaimDimension` field in `Ship`

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
String dim = ship.getChunkClaimDimension();
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
var dim = ship.chunkClaimDimension
```

</tab>
</tabs>

The dimension ID of the chunk claim.

The ship always has to be in the same dimension as the chunk claim, so this is also the dimension ID of the ship.

You can find more information about dimension IDs [here](Dimension-Ids.md)