# How to setup an attachment

Learn how to make a basic attachment class and add it to a ship

## Introduction

Say you want to store data on a particular ship, for example a `boolean` value on whether its in "turbo mode".

To store this data, you need an attachment. ([Why?](The-Attachment-System-Explained.md))

Luckily, setting one up is relatively straightforward.

## Steps

### Setup a public, final class

The attachment class must be public (so it can be registered), and final (so it cannot be extended).

To make a kotlin class final, simply omit the `open` keyword

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class MyAttachment()
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class MyAttachment {
    public MyAttachment() {}
}
</code-block>
</tab>
</tabs>

### Add information to the class

Now that you have your class, lets make it actually useful by adding the `boolean` we want to store.

We can do this by simply storing it in a property and adding a constructor parameter.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class MyAttachment(var turboMode: Boolean)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class MyAttachment {
    public boolean turboMode;

    public MyAttachment(boolean turboMode) {
        this.turboMode = turboMode;
    }
}
</code-block>
</tab>
</tabs>

Keep in mind these values may be accessed on different threads, often the gamethread and/or the physics thread. You may need to make it `volatile` or a `Concurrent` class.

### Registering the class as an attachment

Attempting to add the attachment to a ship will crash if the attachment has not been registered beforehand. 

It's recommended to register your attachments in your mod's constructor.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
import org.valkyrienskies.mod.common.vsCore

fun init() {
    vsCore.registerAttachment(MyAttachment::class.java)
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
public MyMod() {
    ValkyrienSkiesMod.getApi().registerAttachment(MyAttachment.class);
}
</code-block>
</tab>
</tabs>

### Adding the attachment to a ship

> If you want the attachment to be applied to _every_ ship, you can pair the adding of the attachment with the [ShipLoadEvent]().
>
{style="tip"}

Now that you have your attachment class registered and ready to go, it's time to add it to a ship. 

You can **only** add attachments to `LoadedServerShip`s. You may need to use `getLoadedShipManagingPos` instead of `getShipManagingPos`, or do a (safe) cast.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
ship.setAttachment(MyAttachment(true)) // turboMode = true
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ship.setAttachment(new MyAttachment(true)); // turboMode = true
</code-block>
</tab>
</tabs>

### Getting the attachment from a ship

A `LoadedServerShip` is also required for getting your attachment back, for example to _check_ the `turboMode` of a ship.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
ship.getAttachment&lt;MyAttachment&gt;()
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
ship.getAttachment(MyAttachment.class);
</code-block>
</tab>
</tabs>

### Updating the attachment of a ship

To update the data of an attachment, simply get the attachment, change the value you want to change, then add back the attachment with the updated instance.

For example, to toggle the value of our `turboMode`:

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
val attachment = ship.getAttachment&lt;MyAttachment&gt;() 
    // make a new attachment if the ship didn't have one before
    ?: MyAttachment(true) 

attachment.turboMode = !attachment.turboMode
ship.setAttachment(attachment)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
MyAttachment attachment = ship.getAttachment(MyAttachment.class);
if (attachment == null) {
    // make a new attachment if the ship didn't have one before
    attachment = new MyAttachment(true);
}
attachment.turboMode = !attachment.turboMode;
ship.setAttachment(attachment);
</code-block>
</tab>
</tabs>