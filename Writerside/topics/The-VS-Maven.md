# The Valkyrien Skies Maven

To add Valkyrien Skies to your Gradle setup, you will find yourself needing to add the Valkyrien Skies maven.

Usually that looks something like:

<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
repositories {
    // ...
    maven {
		name = 'Valkyrien Skies'
		url = 'https://maven.valkyrienskies.org'
	}
}
</code-block>
</tab>
<tab title="Kts" group-key="kts">
<code-block lang="kotlin">
repositories {
    // ...
    maven {
        name = "Valkyrien Skies"
        url = uri("https://maven.valkyrienskies.org")
    }
}
</code-block>
</tab>
</tabs>

However, there are a few things to be aware of when using the Valkyrien Skies maven.

## Circle CI and what it changes

Valkyrien Skies uses automated builds on [Circle CI](https://circleci.com/) so that every Github commit
is automatically tested and built into a jar.

The purpose of using Circle CI is to provide a clean workspace to build the mod, untainted from any local files a developer may have.

So, when building on Circle CI, the `block_external_repositories` gradle property is set to `true`. This means
Valkyrien Skies can only pull its gradle dependencies from the VS maven and nowhere else.

Because of this, the Valkyrien Skies maven is setup such that it will _mirror_ any dependencies that Valkyrien Skies needs.

> What is mirroring? 
> 
> When a maven _mirrors_ a dependency, it means it copies it over from another maven into its own files (unchanged).
> 
> Essentially cloning the dependency into itself, so that it can also provide the dependency out to people.
> 
{style="tip"}

This leads to the Valkyrien Skies maven including a lot of artifacts that are not actually made by the Valkyrien Skies team.

## How Gradle finds dependencies

When you add a dependency, let's say something like:

<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
implementation 'com.someone:mylibrary:1.0.0' 
</code-block>
</tab>
<tab title="Kts" group-key="kts">
<code-block lang="kotlin">
implementation("com.someone:mylibrary:1.0.0")
</code-block>
</tab>
</tabs>

Gradle will go through every maven defined in your `repositories` block one by one looking for it.

Once it finds a maven that says it has that dependency, it will stop looking, and only pull that dependency from that maven.

Gradle will look through your mavens in a specific order of priority. 
From top to bottom of the `repositories` block, and from the root project inwards.

> When using a multiloader setup where you have multiple gradle files such as 
> `build.gradle`, `common/build.gradle`, `forge/build.gradle`, and `fabric/build.gradle`, 
> the priority of mavens can be a little hard to keep track of. 
> 
> The repositories block in the "root" `build.gradle` will be treated as a higher priority than the subproject (`common`, `fabric`, etc) `build.gradle`'s,
> even if those subprojects explicitly redefine a maven.
> 
> This is important to know, since if you put the VS maven in the root build.gradle, it may take priority over the subprojects.
> 
> This can be a bad thing, for reasons that will be explained later.
>
{style="note"}

## So what's the problem

The problem is that the Valkyrien Skies maven sometimes has _partial_ dependencies.
We'll use `net.minecraftforge:mergetools` as an example, as it demonstrates the problem.

Valkyrien Skies only requires `mergetools-api:1.1.7`. So only that dependency is mirrored to the Valkyrien Skies maven.
However, depending on your gradle setup, Forge itself sometimes requires other artifacts of `mergetools` such as `mergetools-impl:1.1.7`.

If Gradle finds the Valkyrien Skies maven before it finds the Minecraft Forge maven (which would happen if the 
Valkyrien Skies maven is in the top `build.gradle` and the forge maven is only in the `forge/build.gradle`), then it will try
and get all `mergetools` from the Valkyrien Skies maven. It will fail, since the Valkyrien Skies maven only has `-api` and not the other
artifacts that gradle needs.

Instead of searching further, and eventually finding the forge maven which _does_ have the `mergetools` that Gradle needs, 
it simply gives up and stops importing your project.

## How to avoid issues

Luckily you do have a few options available to prevent the Valkyrien Skies maven from messing up your imports.

<procedure title="Whitelist dependencies" id="whitelist-vs-maven">

Only include specific groups from the Valkyrien Skies maven so Gradle only grabs those dependencies

<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
repositories {
    // ...
    maven {
		name = 'Valkyrien Skies'
		url = 'https://maven.valkyrienskies.org'
        content {
            includeGroupAndSubgroups('org.valkyrienskies')
        }
	}
}
</code-block>
</tab>
<tab title="Kts" group-key="kts">
<code-block lang="kotlin">
repositories {
    // ...
    maven {
        name = "Valkyrien Skies"
        url = uri("https://maven.valkyrienskies.org")
        content {
            includeGroupAndSubgroups("org.valkyrienskies")
        }
    }
}
</code-block>
</tab>
</tabs>

</procedure>

<procedure title="Blacklist dependencies" id="blacklist-vs-maven">

Exclude only the groups that fail because of the Valkyrien Skies maven

<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
repositories {
    // ...
    maven {
		name = 'Valkyrien Skies'
		url = 'https://maven.valkyrienskies.org'
        content {
            // Common culprits
            excludeGroup('net.minecraftforge')
            excludeGroup('net.jodah')
            excludeGroup('org.spongepowered')
        }
	}
}
</code-block>
</tab>
<tab title="Kts" group-key="kts">
<code-block lang="kotlin">
repositories {
    // ...
    maven {
        name = "Valkyrien Skies"
        url = uri("https://maven.valkyrienskies.org")
        content {
            // Common culprits
            excludeGroup("net.minecraftforge")
            excludeGroup("net.jodah")
            excludeGroup("org.spongepowered")
        }
    }
}
</code-block>
</tab>
</tabs>

</procedure>

<procedure title="Carefully set maven order" id="order-vs-maven">

> This option isn't recommended, it's difficult to maintain
>
{style="warning"}

Make sure that the Valkyrien Skies maven is always a lower priority than other mavens.

Mavens used in subprojects (e.g. `forge/build.gradle`) should be moved to the root `build.gradle` so that they 
can be above the Valkyrien Skies maven.

Or the Valkyrien Skies maven should be moved to be at the bottom of the 
`repositories` in each subproject (and not be in the root `build.gradle`).


<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
repositories {
    // Every maven in the same file, every maven is above the VS maven
    maven {
        name = 'Forge'
        url = 'https://maven.minecraftforge.net/'
    }
    // ...
    maven {
		name = 'Valkyrien Skies'
		url = 'https://maven.valkyrienskies.org'
	}
}
</code-block>
</tab>
<tab title="Kts" group-key="kts">
<code-block lang="kotlin">
repositories {
    // Every maven in the same file, every maven is above the VS maven
    maven {
        name = "Forge"
        url = uri("https://maven.minecraftforge.net/")
    }
    // ...
    maven {
		name = "Valkyrien Skies"
		url = uri("https://maven.valkyrienskies.org")
	}
}
</code-block>
</tab>
</tabs>

</procedure>