# Dimension IDs
Dimension IDs are slightly different in VS2, so you need to keep that in mind when using vanilla dimension ids

VS Dimension IDs are in the format `minecraft:dimension:[mod]:[dimension name]`.
To convert this to a `ResourceKey<Level>` (which is how Minecraft stores level IDs), you can do this:

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
public static ResourceKey<Level> VSToDimensionKey(String vsDimId) {
    // Split 'minecraft:dimension:namespace:dimension_name' into 
    // [minecraft, dimension, namespace, dimension_name]
    String split = vsDimId.split(":");
    ResourceLocation re = ResourceLocation(split[split.size - 2], split[split.size - 1])
    return ResourceKey.create(Registry.DIMENSION_REGISTRY, rl);
}
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
/* DimensionId in VS2 is a type alias for String */
fun DimensionId.toDimensionKey() {
    // Split 'minecraft:dimension:namespace:dimension_name' into 
    // [minecraft, dimension, namespace, dimension_name]
    val split = this.split(":")
    val rl = ResourceLocation(split[split.size - 2], split[split.size - 1])
    return ResourceKey.create(Registry.DIMENSION_REGISTRY, rl)
}
```

</tab>
</tabs>

This can now be used to, for example, get the `ServerLevel` of a ship when given a `MinecraftServer` object. Something like:

<tabs group="ktj">
<tab title="Java" group-key="java">

```java
public static ServerLevel fromLevelRL(ResourceKey<Level> rl) {
    return server.getLevel(rl);
}
```

</tab>
<tab title="Kotlin" group-key="kotlin">

```kotlin
fun fromLevelRL(rl: ResourceKey<Level?>?): ServerLevel {
    return server.getLevel(rl)
}
```

</tab>
</tabs>