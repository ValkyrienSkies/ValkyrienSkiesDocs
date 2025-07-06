# What's new in VS 2.5

> Developers: see the [migration guide](How-to-migrate-to-VS-2.5.md) for instructions on how to update your addons and
> mods.
>
{style="note"}

**Not yet released**

The Valkyrien Skies 2.5 release is here! Here are the main highlights:

- **PhysX**: The Krunch physics engine is now powered by a modified version of NVIDIA's PhysX, which brings an up to
  **2500%** increase in physics performance, and significantly improved [joints](#new-joints).
- **Minecraft entity interactions**: Players and entities no longer appear to lag behind ships moving at high speed.
  Minecraft's point of interest system now supports ships, meaning that villager jobs, nether portal linking, lightning
  rods, and beehives work as expected.
- **Stable API**: Valkyrien Skies now has a stable public API. Updates to Valkyrien Skies will no longer break addons
  that use only the public API, so addon developers can focus on creating great addons rather than porting to new 
  versions of VS.
- **Documentation**: Official documentation for developers is now available! See the sidebar of this website 
  for the available articles. The KDocs, which are viewable both inside your IDE and at this link, have also been 
  revamped.
- **Compatibility**: VS 2.5 is more compatible with other mods than ever before, and can be dropped into most kitchen
  sink modpacks without issue.


## PhysX 

The Krunch physics backend is now powered by a modified version of NVIDIA PhysX 5.1.

### Crazy Performance 🚀

The physics engine is now able to handle upwards of **5 thousand** ships,  which is **2500%** more than the
approximately two hundred ships that it formerly would choke on.

Having that many ships will still lag your game, but that's now entirely because Minecraft can't handle so many chunks 
loaded at once, and not because of the physics engine.

### New Joints

<secondary-label ref="experimental" />

Todo

## Minecraft entity interactions

