# How to subscribe to VS events

Learn how to subscribe to the custom Valkyrien Skies event system.

## Introduction

Valkyrien Skies has its own loader-independent event system, with many useful events, such as "ship load" or "collision persist".

Usage is relatively straightforward.

## Steps

### Subscribing to the event

To subscribe to the event, you simply register a lambda using the `vsCore` singleton.

This can be done at any point in the mod lifecycle, but is commonly done in the mod constructor.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.common.vsCore

fun init() {
    vsCore.shipLoadEvent.on { event ->
        println("Ship ${event.ship.id} loaded!")
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public MyMod() {
    ValkyrienSkiesMod.getApi().getShipLoadEvent().on((event) -> {
        System.out.println(
            "Ship "+event.getShip().getId()+" loaded!"
        );
    });
}
</code-block>
</tab>
</tabs>

### Abstracting the event

You don't have to pass an anonymous lambda, you can abstract it into a method and pass that instead.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.common.vsCore

class MyMod {
    fun init() {
        vsCore.physTickEvent.on(::physTick)
    }

    private fun physTick(event: PhysTickEvent) {
        println("Physics tick!")
    }
}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyMod {
    public MyMod() {
        ValkyrienSkiesMod.getApi().getPhysTickEvent()
            .on(this::physTick);
    }
    
    private void physTick(PhysTickEvent event) {
        System.out.println("Physics tick!");
    }
}
</code-block>
</tab>
</tabs>

### Unsubscribing from the event

In the rare case you want your listener to _stop_ listening, you can use an overload 
of `.on` which takes two parameters. 

The first parameter will be your event, the second will be a `RegisteredListener`.
This `RegisteredListener` allows you to unsubscribe from the event.

The following example subscribes to the (physTick) event, fires a single time, then unsubscribes itself and will not trigger again.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.common.vsCore

class MyMod {
    fun init() {
        vsCore.physTickEvent.on(::physTick)
    }

    private fun physTick(event: PhysTickEvent, handler: RegisteredListener) {
        println("Single physics tick")
        handler.unregister()
    }
}

</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public class MyMod {
    public MyMod() {
        ValkyrienSkiesMod.getApi().getPhysTickEvent()
            .on(this::physTick);
    }

    private void physTick(PhysTickEvent event, RegisteredListener handler) {
        System.out.println("Single physics tick");
        handler.unregister();
    }
}
</code-block>
</tab>
</tabs>