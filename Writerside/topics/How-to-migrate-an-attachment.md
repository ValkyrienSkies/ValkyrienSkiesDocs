# How to migrate an attachment

Learn how to make a backwards-compatible update to the data stored by your attachment.

## Introduction

Suppose you made a `RocketShipAttachment` for your rocket ship mod, which stores the amount of fuel in a rocket ship. Previously, your attachment stored this data as a `String`, but you've since realized that it's probably better to store it as a `Double`, so you want to migrate it.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class RocketShipAttachment(var fuel: String)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class RocketShipAttachment {
    private String fuel;

    public RocketShipAttachment(String fuel) {
        this.fuel = fuel;
    }

    public void setFuel(String fuel) {
        this.fuel = fuel;
    }

    public String getFuel() {
        return fuel;
    }
}
</code-block>
</tab>
</tabs>


## Steps

### Identify the serialization key for your previous attachment

Find the serialization `key` you specified in the original `AttachmentRegistration`. If you did not specify a `key`, it defaults to the [fully qualified name](https://docs.oracle.com/javase/specs/jls/se10/html/jls-6.html#jls-6.7) of the attachment class. In our case, that's `my.packagename.RocketShipAttachment`.

### Rename your old attachment class, and then create the new one

You will need to keep your old attachment class, but you can rename it. We'll rename it to `RocketShipAttachmentV1`.

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
package my.packagename

class RocketShipAttachmentV1(var fuel: String)
class RocketShipAttachment(var fuel: Double)
</code-block>
</tab>
<tab title="Java" group-key="java">
<code-block lang="Java">
package my.packagename;

public final class RocketShipAttachmentV1 {
    private String fuel;

    public RocketShipAttachmentV1(String fuel) {
        this.fuel = fuel;
    }

    public void setFuel(String fuel) {
        this.fuel = fuel;
    }

    public String getFuel() {
        return fuel;
    }
}

public final class RocketShipAttachment {
    private double fuel;

    public RocketShipAttachment(double fuel) {
        this.fuel = fuel;
    }

    public void setFuel(double fuel) {
        this.fuel = fuel;
    }

    public double getFuel() {
        return fuel;
    }
}

</code-block>
</tab>
</tabs>

### Update your attachment registrations

<tabs group="ktj">
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
vsApi.registerAttachment(RocketShipAttachmentV1::class.java) {
    // Explicitly specify the serialization key,
    // since we renamed the class
    key = "my.packagename.RocketShipAttachment"

    useUpdater(RocketShipAttachment::class.java) { v1 ->
        // Convert the v1 attachment into the new version
        RocketShipAttachment(v1.fuel.toDouble())
    }
}

vsApi.registerAttachment(RocketShipAttachment::class.java) {
    // We need a new, unique serialization key, 
    // so we'll append :2 to the end of the original key
    key = "my.packagename.RocketShipAttachment:2"
}
</code-block>
</tab>
<tab title="Java" group-key="java">
<!--suppress WrsCodeBlockWidthInspection -->
<code-block lang="Java">
AttachmentRegistration r1 = ValkyrienSkies.api()
    .newAttachmentRegistrationBuilder(RocketShipAttachmentV1.class)
    // Explicitly specify the serialization key,
    // since we renamed the class
    .key("my.packagename.RocketShipAttachment")
    .useUpdater(
        RocketShipAttachment.class,
        // Convert the v1 attachment into the new version
        v1 -> new RocketShipAttachment(Double.parseDouble(v1.getFuel()))
    )
    .build();
ValkyrienSkies.api().registerAttachment(r1);

AttachmentRegistration r2 = ValkyrienSkies.api()
    .newAttachmentRegistrationBuilder(RocketShipAttachment.class)
    // We need a new, unique serialization key,
    // so we'll append :2 to the end of the original key
    .key("my.packagename.RocketShipAttachment:2")
    .build();
ValkyrienSkies.api().registerAttachment(r2);
</code-block>
</tab>
</tabs>

