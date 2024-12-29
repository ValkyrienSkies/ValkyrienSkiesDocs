<show-structure depth="2"/>

# How to migrate to VS 2.5

Valkyrien Skies 2.5 introduced a number of breaking API changes. Addon developers and other mod developers with
VS integration will need to update their mods.

> **Are you a player or modpack developer?**
>
> You might need to update your addons to the latest version, or wait until their authors update them to be compatible
> with VS 2.5
>
{style="tip"}

## Required changes

These changes are required for your mod or addon to continue functioning with VS 2.5.

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

> Try to avoid using `apigame` - we don't make any [backwards compatibility](Compatibility.md#backwards-compatibility) 
> guarantees for it.
> 
{style="warning"}

Constraints are now [joints](Joints.md), and everything in `org.valkyrienskies.core.apigame.constraints` has been 
removed. 


### Use `rebuild` and `toBuilder` instead of `ShipTransform.copy`

> Also recommended: start using `BodyTransform` instead of `ShipTransform`. You can freely cast between the two.
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

## Recommended changes

These changes aren't strictly required, but they will make your life easier during future updates and improve the
quality of your code.

### Use recommended packages

**For Java users:** Start using `ValkyrienSkies`. Stop using `VSGameUtilsKt` and `VectorConversionsMCKt`. Remove  
imports from `org.valkyrienskies.mod.common`. 

**For Kotlin users:** Remove imports from `org.valkyrienskies.mod.common`.

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