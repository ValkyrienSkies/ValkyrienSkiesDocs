# The Shipyard

The **shipyard** is a far off region of the world where ships' chunks are stored. Ships consist of chunks in the 
shipyard that are "projected" into their world location.

> Currently, the shipyard is located at the coordinates $-28672000 \leq x \leq 28672000$ and 
> $12288000 \leq z \leq 28672000$, but don't depend on it.

## Why a shipyard?

There are other ways add movable ships to Minecraft, but they all have deficiencies compared to the shipyard.

### Custom Entities

Mods like [Archimedes' Ships](https://www.curseforge.com/minecraft/mc-mods/archimedes-ships) and its successors 
([DaVinci's Vessels](https://www.curseforge.com/minecraft/mc-mods/davincis-vessels), 
[Plato's Transporters](https://www.curseforge.com/minecraft/mc-mods/platos-transporters), etc.) use custom entities
to represent ships, and store each ship's blocks using NBT tags stored with the entity.

The [Create](https://www.curseforge.com/minecraft/mc-mods/create) mod also uses custom entities to represent its 
contraptions.

**Pros:**
- **Simplicity:** Custom entities typically don't require much code and are easy to port to newer versions.

**Cons:**
- **Not functional:** Blocks on ships can't be used or ticked, so one can't craft, sleep, use chests, doors, redstone,
vegetable farms, mob farms, etc. on ships - most of the things one expects to do while playing Minecraft.
- **Incompatible:** Not even vanilla blocks are functional on ships, so modded blocks don't stand a chance.
- **Bad performance:** Minecraft entities have notoriously bad performance. Furthermore, these mods are generally forced 
to reimplement Minecraft chunk rendering, often with worse performance than vanilla and especially worse than
optimization mods like [Sodium](https://www.curseforge.com/minecraft/mc-mods/sodium).

### Separate Dimensions

[Immersive Portals](https://www.curseforge.com/minecraft/mc-mods/immersive-portals-mod) is the only mod that can
project chunks from one dimension to another. 

> ***Definition*** - **Dimension:** A Minecraft dimension, like the Overworld or Nether, represented by the `Level` class.

**Pros:**
- **Powerful and flexible:** Ships can have unlimited size, because they don't have to share a shipyard, and ships 
automatically work cross-dimensionally. Blocks function perfectly on ships, as long as the game is modified to be
aware of their true dimension and their true position.

> ***Definition*** - **True position:** the position of a block as it appears to the player, rather than its position 
> in the shipyard. In the rest of our documentation, we refer to this as the "world" or "world space" position.
>
> ***Definition*** - **True dimension:** the dimension of a block as it appears to the player, rather than the dimension
> of its  shipyard.

**Cons:**
- **Complicated and incompatible:** A substantial amount of Minecraft code and other mods expect blocks to be situated 
in the same dimension as the entities and other blocks that they interact with. The Minecraft client also expects there 
to be only a single dimension loaded at all times. This means that Immersive Portals has a huge amount of invasive 
mixins, and is subsequently incompatible with many other mods and difficult to update. Furthermore, mixins that 
patch the game to be aware of block's true dimensions can be complex and difficult to write.
- **Bad performance:** Loading multiple dimensions (which involves loading chunks, in addition to other things required
for each dimension) has more overhead than *only* loading chunks.

### Shipyard

Valkyrien Skies uses a shipyard to store ships. Each ship has a chunk claim in the shipyard, and each ship
stores its blocks in "ship chunks" inside the chunk claim, in the same dimension as the ship.

**Pros:**
- **Powerful and flexible (relatively)**: Ships can be as large as the plots. Blocks function perfectly on ships, as 
long as they are aware of their true position.
- **Compatible (relatively)**: Blocks may work incorrectly if the game is unaware of their true position, but patches
to change position are generally much easier to write and maintain than patches to change dimensions.
> **Ackshually... Valkyrien Skies is incompatible with X, Y and Z!**
>
> Valkyrien Skies does occasionally have compatibility problems. Even so, we believe the shipyard system is the best. 
> 
> Consider this: even if every single block in the shipyard is completely nonfunctional due to an incompatibility, 
> the shipyard is still working just as well as the entity-based systems.
{style="note"}
- **Good performance**: The overhead of ships is the same as the overhead of loading more chunks. Ships can benefit
from existing performance mods like Lithium and Sodium.

**Cons:**
- **Not as powerful as separate dimensions**: Ships can't become larger than their plots.

## Chunk claims

In the shipyard, each ship is assigned a **chunk claim** inside the shipyard. In VS2, each chunk claim is 256x256 
chunks. The chunks in the chunk claim are then "projected" into the world using the corresponding ship's transform.