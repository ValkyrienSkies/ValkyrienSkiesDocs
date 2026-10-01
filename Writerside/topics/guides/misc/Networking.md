# Networking

Valkyrien Skies uses its own loader-abstracted networking system, which addons can use too.

This means your networking can be the same on both forge and fabric easily, without libraries like Architectury API (or your own abstractions).

## Setting up your packet class

First, you need a packet class. This class just needs to store the data that is being sent across the network.
It should extend `SimplePacket`, no matter the direction it will go (server -> client or client -> server).

(The integer data is an example, the data can be multiple properties and can be different types)

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
data class MyPacket(val myData: Int): SimplePacket
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public record MyPacket(int myData) implements SimplePacket {}
</code-block>
</tab>
</tabs>

## Registering your packet

To use your packet, it must be registered to Valkyrien Skies. 

> Valkyrien Skies doesn't actually care when you register your packets, as long as it is done before you attempt to use said packet.
> 
> However, it's _recommended_ to do it in your main mod initialization.
> 
{style="note"}

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
with(vsCore.simplePacketNetworking) {
    MyPacket::class.register()
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ValkyrienSkiesMod.getVsCore().getSimplePacketNetworking().register(
    JvmClassMappingKt.getKotlinClass(MyPacket.class)
);
</code-block>
</tab>
</tabs>

## Adding listeners

This is where you specify whether your packet is server -> client (s2c) or client -> server (c2s). 

The listener setup is very similar to the initial packet registration. 

You have the option to register a server handler (for a c2s packet) or a client handler (for a s2c packet).

The server listener receives a `VsiPlayer`, representing the client which sent the packet.
Both listener types receive an instance of the packet class itself (after it has been deserialized)

`VsiPlayer` is an abstraction of the Minecraft `Player` that VS-Core uses to stay Minecraft agnostic.
You can obtain a normal `Player` from the `VsiPlayer` using:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val player = (vsiPlayer as MinecraftPlayer).player as ServerPlayer
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ServerPlayer player = (ServerPlayer) ((MinecraftPlayer) vsiPlayer).getPlayer();
</code-block>
</tab>
</tabs>

To register your listeners:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
with(vsCore.simplePacketNetworking) {
    MyToServerPacket::class.registerServerHandler { packet, vsiPlayer -> 
        // Do stuff (on server)
    }
    
    MyToClientPacket::class.registerClientHandler { packet -> 
        // Do stuff (on client)
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ValkyrienSkiesMod.getVsCore().getSimplePacketNetworking().registerServerHandler(
    JvmClassMappingKt.getKotlinClass(MyToServerPacket.class),
    (packet, vsiPlayer) -> {
        // Do stuff (on server)
        return null;
    }
);

ValkyrienSkiesMod.getVsCore().getSimplePacketNetworking().registerClientHandler(
    JvmClassMappingKt.getKotlinClass(MyToClientPacket.class),
    (packet) -> {
        // Do stuff (on client)
        return null;
    }
);
</code-block>
</tab>
</tabs>

## Sending your packets

Now you've got a packet registered, and you've got it running code when it is recieved.
How do you actually send it out?

Here's how you would send a packet from a client to the server:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
with(vsCore.simplePacketNetworking) {
    // We put 0 here because our example MyPacket
    // from earlier stored an integer as its data
    MyPacket(0).sendToServer()
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ValkyrienSkiesMod.getVsCore().getSimplePacketNetworking().sendToServer(
    // We put 0 here because our example MyPacket
    // from earlier stored an integer as its data    
    new MyPacket(0)
);
</code-block>
</tab>
</tabs>

And here's from the server to a specific client:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
// The client to send the packet to
val player: Player = // ...
with(vsCore.simplePacketNetworking) {
    // We put 0 here because our example MyPacket
    // from earlier stored an integer as its data
    MyPacket(0).sendToClient(player.playerWrapper)
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
// The client to send the packet to
Player player = // ...
ValkyrienSkiesMod.getVsCore().getSimplePacketNetworking().sendToClient(
    // We put 0 here because our example MyPacket
    // from earlier stored an integer as its data    
    new MyPacket(0),
    ((PlayerDuck) player).vs_getPlayer()
);    
</code-block>
</tab>
</tabs>

There's also `sendToClients` and `sendToAllClients` but those should hopefully be self-explanatory.

## How serialization works

The Valkyrien Skies networking uses a `CBORMapper` Jackson Serializer, which is configured to be able to serialize:
- All primitive types (numbers, strings, maps, lists, etc)
- Data/record classes
- All JOML types (Vector3d, Quaterniond, etc)
- ShipData and VsBodyData

The mapper will also ignore (not serialize/deserialize) any property annotated with `@JsonIgnore`

Keep in mind that Minecraft classes (e.g. `BlockPos` or `Vec3`) can't be serialized. 
Often they can be converted to/from a lower primitive. 

For example `BlockPos` can do the following:
<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val pos: BlockPos = // ...
val posToLong = pos.asLong()
// Send the long through the packet
val posFromLong = BlockPos.of(posToLong)
assert(pos == posFromLong) // always true
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
BlockPos pos = // ...
long posToLong = pos.asLong();
// Send the long through the packet
BlockPos posFromLong = BlockPos.of(posToLong);
assert pos == posFromLong; // always true
</code-block>
</tab>
</tabs>

Note that at the moment, it is not possible to add custom serializers/deserializers to the Valkyrien Skies networking mapper.