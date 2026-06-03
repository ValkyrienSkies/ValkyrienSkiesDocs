# How to use a physics tick attachment

> Recommended pre-requisite to this guide: [](How-to-setup-an-attachment.md)
>
{style="note"}

While attachments can simply be standalone classes holding data, there is another special use for attachments.

Using the `ShipPhysicsListener` interface, you can have your attachment run logic on every Physics Tick.

A `ShipPhysicsListener` attachment is one of the few ways to get a `PhysShip` or `PhysLevel`.

### Usage

Modify your existing attachment class to implement the ShipPhysicsListener interface

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class MyAttachment(): ShipPhysicsListener
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class MyAttachment implements ShipPhysicsListener {
    public MyAttachment() {}
}
</code-block>
</tab>
</tabs>

Then implement the physics tick method:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class MyAttachment(): ShipPhysicsListener {
    override fun physTick(
        physShip: PhysShip, physLevel: PhysLevel
    ) {
        // do stuff
    }
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class MyAttachment implements ShipPhysicsListener {
    public MyAttachment() {}

    @Override
    public void physTick(@NotNull PhysShip physShip, @NotNull PhysLevel physLevel) {
        // do stuff
    }
}
</code-block>
</tab>
</tabs>

### Being wary of threads

> This section is important! If you aren't careful, you could end up with random crashes.
>
{style="warning"}

The physics tick method is called on a different thread to the normal minecraft game thread. This means that 
any values you access from **both** the physics tick **and** the game tick must be thread safe! 

For example, if my mod updates a `direction` variable in the attachment every game tick (e.g. in a Block Entity tick method),
**and** it reads that variable in the `physTick` method, that variable must be thread safe.

Usually that means making it `volatile` or a `Concurrent` class.

If you ignore this advice, the game could crash randomly with a `ConcurrentModificationException`.
If your users are experiencing this crash, check your attachments, you might've missed making a variable thread safe.

> **Under no circumstances should you store a `Level` (or equivalent) object in an attachment!**
> All the `Level` methods are for the game tick only. You should call them on game tick, then store the _result_ in your attachment.
>
{style="warning"}