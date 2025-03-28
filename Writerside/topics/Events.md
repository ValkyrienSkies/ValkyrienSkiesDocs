
# Events
VS2 provides some useful events in `VSEvents` and `VSGameEvents`

## All events

`org.valkyrienskies.core.impl.hooks.VSEvents`:

| Name | Description | Arguments |
| - | - | - |
| `shipLoadEvent` | Will be called once a ship gets (chunk) loaded on the server | `ship: ShipObjectServer` |
| `shipLoadEventClient` | Will be called once a ship gets (chunk) loaded on the client | `ship: ShipObjectClient` |
| `tickEndEvent` | Will be called at the end of the server ticking a world | `world: ShipObjectServerWorld` |


`org.valkyrienskies.mod.common.hooks.VSGameEvents`:

| Name | Description | Arguments                                  |
| - | - |--------------------------------------------|
| `registriesCompleted` | Will be called once the mass & similar registries are filled with data |                                            |
| `tagsAreLoaded` | Will be called after Minecraft tags are loaded |                                            |
| `renderShip` | Will be called before a ship is rendered | `event: VSGameEvents.ShipRenderEvent`      |
| `postRenderShip` | Will be called after a ship is rendered | `event: VSGameEvents.ShipRenderEvent`      |
| `shipsStartRendering` | Will be called before all ships are rendered | `event: VSGameEvents.ShipStartRenderEvent` |

## How to listen to events
There is a `on` method in all of those events. Register event listeners to the event through this method.
Its recommended to do this in your mods on-load (init) method. Use it like so:

<tabs>
<tab title="Java">

```java
[DesiredEvent].Companion.on((event) -> {
    // ...
});
```

</tab>
<tab title="Kotlin">

```kotlin
[DesiredEvent].on {(event) -> {
    // ...
}}
```

</tab>
</tabs>

## Example
Print the slug (name) of every ship in a world every tick:

<tabs>
<tab title="Java">

```java
VSEvents.TickEndEvent.Companion.on((e) -> {
    e.getWorld().getAllShips().forEach((ship) -> {
        System.out.println(ship.getSlug());
    });
});
```

</tab>
<tab title="Kotlin">

```kotlin
VSEvents.tickEndEvent.on { e ->
    e.world.allShips.forEach { ship ->
        println(ship.slug)
    }
}
```

</tab>
</tabs>