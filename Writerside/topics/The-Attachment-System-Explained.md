
# The Attachment System

> Much of the attachment system is marked with `@VsBeta` and may change in the future.
> 
{style="note"}


The **attachment system** is a part of vs-core which addons and third parties can use to attach data to ships and other
objects managed by vs-core.

## Use cases

- You want to associate some data with a ship temporarily
- You want to persist/serialize data with a ship
- You want to apply forces to a ship

## Saving, Loading and removing

In the package `org.valkyrienskies.core.api.ships`
there are two extension functions for saving and loading attachments.

To save an attachment you can do: 

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
    serverShip.saveAttachment(MyAttachmentClass.class, new MyAttachmentClass());
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
    serverShip.saveAttachment(MyAttachmentClass())
```

</tab>
</tabs>

And to load the attachment:

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
var attachment = serverShip.getAttachment(MyAttachmentClass.class);
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
val attachment = serverShip.getAttachment<MyAttachmentClass>()
```

</tab>
</tabs>

> The attachment class needs to have a constructor without arguments.
> You can use a constructor with arguments for `saveAttachment` if you wish,
> but you still need one without arguments.
> 
{style="note"}

And finally, to remove the attachment:

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
serverShip.saveAttachment(MyAttachmentClass.class, null);
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
serverShip.saveAttachment<MyAttachmentClass>(null)
```

</tab>
</tabs>

## Attachment class fields
Every class you use as an attachment needs to follow these rules:

- All fields that you don't want to be saved in the attachment need to be marked
  with `@com.fasterxml.jackson.annotation.JsonIgnore`. This includes ships because otherwise they would recursively save themselves.
- All types of the fields that are not marked with
  `@com.fasterxml.jackson.annotation.JsonIgnore` need to be classes with a default
  constructor without arguments. This means that you can't use interfaces as types of these fields

## World "corruption"
If you change anything related to fields that get saved in a ship attachment class,
you will get an error when loading ships with the old data. This is because the ship attachment
class is not compatible with the old data anymore. To fix this, you should create a new ship attachment class
and make the old one copy everything over to the new one, and then delete the old one from a ship.

To migrate or remove old attachments, 
you can register an event listener to migrate ships as soon as they are loaded. 
Simply hook into the ShipLoadEvent in your mods on-load (init) method:

> For ship force inducers, you can also use the `applyForces` method as your event instead.
>
{style="tip"}

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
VSEvents.ShipLoadEvent.Companion.on((shipLoadEvent) -> {
    ServerShip serverShip = shipLoadEvent.getShip();
    // Update / remove your attachments here
});
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
VSEvents.shipLoadEvent.on {(serverShip) -> {
    // Update / remove your attachments here
}}
```

</tab>
</tabs>


> For more information about events in VS2, go to [Events](Events.md)
> 
{style="tip"}

## Force inducers
To apply forces to a ship, you need a force inducer class. This class will extend the `ShipForcesInducer` interface,
and override the `applyForces(PhysShip)` method. Then, once you save this class as an attachment to a ship, the `applyForces`
method will be called each physics tick. You can then use the PhysShip object in `applyForces` to apply forces to your ship.

The `ShipForcesInducer` interface class:
```kotlin
package org.valkyrienskies.core.api.ships

import org.valkyrienskies.core.api.ships.PhysShip
import org.valkyrienskies.core.api.ships.properties.ShipId

@Deprecated("sus")
interface ShipForcesInducer {
    fun applyForces(
        physShip: PhysShip
    )

    fun applyForcesAndLookupPhysShips(
        physShip: PhysShip,
        lookupPhysShip: (ShipId) -> PhysShip?
    ) {
        // Default implementation to not break existing implementations
    }
}
```
{collapsible="true" collapsed-title="ShipForcesInducer.kt"}

> VS2 physic ticks run in a different thread than the minecraft server thread.
> This means that you need to be careful when accessing the same fields in applyForces and elsewhere
> 
{style="warning" title="Thread Safety"}

## Examples

### Ship fuel tracking
<tabs group="ktj">
<tab title="Java" group-key="java">

```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import org.valkyrienskies.core.api.ships.ServerShip;
import org.valkyrienskies.core.impl.api.ServerShipUser;

public class ShipFuelStorage {
    @JsonIgnore
    private ServerShip ship = null;
    int fuel = 0;

    public ShipFuelStorage() {}

    public ShipFuelStorage(ServerShip ship) {
        this.ship = ship;
    }

    public static ShipFuelStorage getOrCreate(ServerShip ship) {
        ShipFuelStorage attachment = ship.getAttachment(ShipFuelStorage.class);
        if (attachment == null) {
            attachment = new ShipFuelStorage(ship);
            ship.saveAttachment(ShipFuelStorage.class, attachment);
        }
        return attachment;
    }
}
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
import com.fasterxml.jackson.annotation.JsonIgnore
import org.valkyrienskies.core.api.ships.ServerShip
import org.valkyrienskies.core.impl.api.ServerShipUser

class ShipFuelStorage(
    @JsonIgnore
    var ship: ServerShip? = null
) {
    var fuel: Int = 0

    companion object {
        fun getOrCreate(ship: ServerShip): ShipFuelStorage =
            ship.getAttachment<ShipFuelStorage>()
                ?: ShipFuelStorage(ship).also {
                    ship.saveAttachment(it)
                }
    }
}
```

</tab>
</tabs>


### Ship thruster force inducer

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
import org.valkyrienskies.core.api.ships.PhysShip;
import org.valkyrienskies.core.api.ships.ShipForcesInducer;

public class ShipThrusterController implements ShipForcesInducer {

    @Override
    public void applyForces(@NotNull PhysShip physShip) {
        // go through each thruster on ship and apply force
    }

    public static ShipThrusterController getOrCreate(ServerShip ship) {
        ShipThrusterController attachment = ship.getAttachment(ShipThrusterController.class);
        if (attachment == null) {
            attachment = new ShipThrusterController(ship);
            ship.saveAttachment(ShipThrusterController.class, attachment);
        }
        return attachment;
    }
}
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
import org.valkyrienskies.core.api.ships.PhysShip
import org.valkyrienskies.core.api.ships.ShipForcesInducer

class ShipThrusterController: ShipForcesInducer {

    override fun applyForces(physShip: PhysShip) {
        // go through each thruster on ship and apply force
    }

    companion object {
        fun getOrCreate(ship: ServerShip): ShipThrusterControler =
            ship.getAttachment<ShipThrusterControler>()
                ?: ShipThrusterControler().also {
                    ship.saveAttachment(it)
                }
    }
}
```

</tab>
</tabs>


