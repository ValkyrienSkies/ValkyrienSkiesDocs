<show-structure depth="2"/>

# How to migrate to VS 2.4

Valkyrien Skies 2.4 introduced a number of breaking API changes. Addon developers and other mod developers with
VS integration will need to update their mods.

> **Are you a player or modpack developer?**
>
> You might need to update your addons to the latest version, or wait until their authors update them to be compatible
> with VS 2.4
>
{style="tip"}

## Required changes

These changes are required for your mod or addon to continue functioning with VS 2.4.

> **Look out for methods and classes marked `@VsBeta` or `@Experimental`**
>
> These APIs (such as the attachments system) are not finalized and might be changed in the future.
>
{style="note"}

### Migrate to the new attachments system

#### Change your attachment imports

> **Be aware of the difference between `saveAttachment` and `setAttachment`**
> 
> In the legacy attachment system, the `saveAttachment` function would serialize the attachment, while the 
> `setAttachment` function would not serialize it. 
> 
> In the new attachment system, there is only `setAttachment`, and the serialization behavior is configured when the
> attachment is registered. If you don't want to serialize your attachment, call `useTransientSerializer()` when calling 
> `registerAttachment()` 
> 
{style="warning"}

The extension functions in `org.valkyrienskies.core.api.ships.*` have been
moved to `org.valkyrienskies.core.api.attachments.*`.

| Original                                           | Replacement                                             |
|----------------------------------------------------|---------------------------------------------------------|
| `org.valkyrienskies.core.api.ships.saveAttachment` | `org.valkyrienskies.core.api.attachments.setAttachment` |
| `org.valkyrienskies.core.api.ships.setAttachment`  | `org.valkyrienskies.core.api.attachments.setAttachment` |
| `org.valkyrienskies.core.api.ships.getAttachment`  | `org.valkyrienskies.core.api.attachments.getAttachment` |

#### Get and set attachments on `LoadedServerShip`, not `ServerShip`

Attachments on potentially unloaded ships are no longer supported.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Kotlin">
val ship: ServerShip = ...
ship.saveAttachment(MyAttachment())
</code-block>
<code-block lang="Kotlin">
val ship: ServerShip = ...
if (ship is LoadedServerShip) { 
   ship.setAttachment(MyAttachment())
}
</code-block>
</compare>
</tab>
<tab title="Java" group-key="java">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Java">
ServerShip ship = ...
ship.saveAttachment(new MyAttachment())
</code-block>
<code-block lang="Java">
ServerShip ship = ...
if (ship instanceof LoadedServerShip loadedShip) {
    loadedShip.setAttachment(new MyAttachment());
}
</code-block>
</compare>
</tab>
</tabs>

#### Register your attachments

Your attachments now need to be registered.

To migrate legacy attachments in a backwards compatible way, make sure to call `useLegacySerializer()` while registering
the attachments.

> If you previously used `setAttachment` to set attachments without serializing them, call `useTransientSerializer()` 
> instead.
> 
{style="note"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
vsApi.registerAttachment(MyAttachment::class.java) {
   useLegacySerializer()
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
AttachmentRegistration registration = ValkyrienSkies.api()
    .newAttachmentRegistrationBuilder(MyAttachment.class)
    .useLegacySerializer()
    .build();

ValkyrienSkies.api().registerAttachment(registration);
</code-block>
</tab>
</tabs>

#### Make your attachments final

Attachment classes are now required to be final - meaning they can't be interfaces and they can't be extended.

> **Final classes are the default in Kotlin**
> 
> You might not need to change anything.
> 
{style="tip"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
class MyAttachment {
   // ...
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
final class MyAttachment {
   // ...
}
</code-block>
</tab>
</tabs>

### Migrate from constraints to joints

> Try to avoid using `internal` - we don't make any [backwards compatibility](Compatibility.md#backwards-compatibility) 
> guarantees for it.
> 
{style="warning"}

Constraints are now [joints](Joints.md), and everything in `org.valkyrienskies.core.apigame.constraints` has been 
removed.

### Migrate to new force application methods

Force application has been revamped. The old methods in `org.valkyrienskies.core.api.ships.PhysShip` have been marked as deprecated, and point to their new counterparts.

| Original                                                                             | Replacement                                                                     |
|--------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| `applyInvariantForce(force)` &&<br><\br>`applyInvariantForceToPos(force, pos)`       | `applyWorldForce(force, pos)` OR<br><\br>`applyWorldForceToBodyPos(force, pos)` |
| `applyInvariantTorque(torque)`                                                       | `applyWorldTorque(torque)`                                                      |
| `applyRotDependentForce(force)` &&<br><\br>`applyRotDependentForceToPos(force, pos)` | `applyBodyForce(force, pos)`                                                    |
| `applyRotDependentTorque(torque)`                                                    | `applyBodyTorque(torque)`                                                       |

> Note: The old methods all behaved as if their positions were in Body Space, meaning they used positions relative to the ship's Center of Mass.
> There are several new methods, which are named according to the reference space they take as positions:
> 
> **[WORLD SPACE]** : Positions are in world coordinates, and directions are relative to the world axes; up is always relative to the sky and typically inverse of gravity.
> 
> **[MODEL SPACE]** : Positions are in shipyard coordinates, and directions are relative to the ship's axes; up is relative to the ship's orientation.
> 
> **[BODY SPACE]** : Positions are relative to the ship's Center of Mass, and directions are relative to the ship's axes; up is relative to the ship's orientation.
> 
> To achieve the effect of the old INVARIANT methods, use methods that apply a WORLD relative force/torque. These are `applyWorldForce`, `applyWorldTorque`, `applyWorldForceToModelPos`, and `applyWorldForceToBodyPos`.
> 
> To achieve the effect of the old ROTATION-DEPENDENT methods, use methods that apply a BODY or MODEL relative force/torque. These are `applyBodyForce`, `applyBodyTorque`, `applyModelForce`, and `applyModelTorque`.
>
{style="note"}

Additionally, there are new methods to apply forces/torques in ship-space coordinates; you can find all of these methods and further documentation in the javadocs for `PhysShip`.

### Migrate from ShipForceAppliers to ShipPhysicsListeners

> Try to avoid using `internal` - we don't make any [backwards compatibility](Compatibility.md#backwards-compatibility)
> guarantees for it.
>
{style="warning"}

The `ShipForceApplier` interface has been removed. Use `ShipPhysicsListener` instead, which takes advantage of the new [PhysLevels](PhysLevels.md) to provide a more cohesive way to access things that reside on the Physics Thread.

`applyForces` and `applyForcesToNearbyShips` have been replaced with `physTick(physShip, physLevel)`. This allows you to add/update/remove joints, enable and disable collision pairs, and access other ships through the provided PhysLevel.
Experimental API features such as retrieving all joints in the world can be accessed through `VsiPhysLevel`, though this is not advised for general use, as it may change in future versions.

## Recommended changes

These changes aren't strictly required, but they will make your life easier during future updates and improve the
quality of your code.

### Use `rebuild` and `toBuilder` instead of `ShipTransform.copy`

> Also recommended: start using `BodyTransform` instead of `ShipTransform`. You can always downcast `BodyTransform` to
> `ShipTransform` if you need to.
>
{style="note"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Kotlin">
val transform: ShipTransform = ...
val newTransform: ShipTransform = transform.copy(
    positionInWorld = Vector3d(123)
)
</code-block>
<code-block lang="Kotlin">
val transform: ShipTransform = ...
val newTransform: BodyTransform = transform.rebuild {
    position.set(Vector3d(123))
}
</code-block>
</compare>
</tab>
<tab title="Java" group-key="java">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Java">
ShipTransform transform = ...
ShipTransform newTransform = transform.copy(
    new Vector3d(123), 
    transform.getPositionInShip(),
    transform.getRotation(),
    transform.getScaling()
);
</code-block>
<code-block lang="Java">
ShipTransform transform = ...
BodyTransform newTransform = transform.toBuilder()
    .position(new Vector3d(123))
    .build();
</code-block>
</compare>
</tab>
</tabs>

### Use recommended packages

**Java users:** 
- Use `ValkyrienSkies` instead of `VSGameUtilsKt` and `VectorConversionsMCKt`. 
- Remove imports from `org.valkyrienskies.mod.common`. 

**Kotlin users:** 
- Remove imports from `org.valkyrienskies.mod.common`.

> **Check the deprecation warnings**
> 
> Each deprecated function or class will also have a suggested replacement. You can check it by hovering your mouse
> over the name of the deprecated identifier.
> 
> In Kotlin, IntelliJ IDEA can automatically make the replacement.
> 
{style="note"}

#### Use `getShipManagingBlock`/`getShipManagingChunk` instead of `getShipManagingPos`

`getShipManagingPos` has been renamed and moved. We'll keep the old versions around, but try not to use them.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.common.getShipManagingPos

val level: Level = ...
val pos: BlockPos = ...
val ship = level.getShipManagingPos(level, pos)
</code-block>
<code-block lang="Kotlin">
import org.valkyrienskies.mod.api.getShipManagingBlock

val level: Level = ...
val pos: BlockPos = ...
val ship = level.getShipManagingBlock(level, pos)
</code-block>
</compare>
</tab>
<tab title="Java" group-key="java">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Java">
import org.valkyrienskies.mod.common.VSGameUtilsKt

Level level = ...
BlockPos pos = ...
Ship ship = VSGameUtilsKt.getShipManagingPos(level, pos);
</code-block>
<code-block lang="Java">
import org.valkyrienskies.mod.api.ValkyrienSkies

Level level = ...
BlockPos pos = ...
Ship ship = ValkyrienSkies.getShipManagingBlock(level, pos);
</code-block>
</compare>
</tab>
</tabs>

#### Use `vsApi.xxEvent` instead of `VSEvents.XxEvent`

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Kotlin">
VSEvents.ShipLoadEvent.on { ev -> }
</code-block>
<code-block lang="Kotlin">
vsApi.shipLoadEvent.on { ev -> }
</code-block>
</compare>
</tab>
<tab title="Java" group-key="java">
<compare type="top-bottom" first-title="Before" second-title="After">
<code-block lang="Java">
VSEvents.ShipLoadEvent.Companion.on(ev -> {});
</code-block>
<code-block lang="Java">
ValkyrienSkies.api().getShipLoadEvent().on(ev -> {});
</code-block>
</compare>
</tab>
</tabs>
