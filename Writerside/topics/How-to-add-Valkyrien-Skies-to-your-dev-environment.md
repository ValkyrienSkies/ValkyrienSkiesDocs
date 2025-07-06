
<link-summary>
In this guide, we'll add Valkyrien Skies to an existing development environment or template mod.
</link-summary>

# How to add Valkyrien Skies to your dev environment

> If you're starting a new addon and don't have an existing development environment, see [](Develop-you-first-addon.md)
>
{style="note"}

In this guide, we'll add Valkyrien Skies to an existing development environment or template mod.

## Before you start

Make sure that you:

- Have an existing Minecraft mod development environment.
- Have [installed the prerequisites for addon development](How-to-install-the-prerequisites-for-addon-development-on-Windows.md).

It's recommended that you:

- Use IntelliJ IDEA.
- Do not use Eclipse, Visual Studio Code or any other IDE. 
  > IDEs other than IntelliJ IDEA have poor support for Minecraft modding gradle plugins, Kotlin, and are uncommon in
  > the modding community, which will make your life difficult. 


## How to add Valkyrien Skies to ForgeGradle

1. Open your `build.gradle` (Groovy) or `build.gradle.kts` (Kotlin) file.
2. <include from="snippets.md" element-id="gradle_add_repository"/>
3. Find the `dependencies` block and add Valkyrien Skies as a dependency. It should look like this:
    <tabs group="ktg">
    <tab title="Groovy" group-key="groovy">
    <code-block lang="Groovy">
    dependencies {
        // ... your other dependencies
        implementation("%maven_version%") {
            transitive = false
        }
    }
    </code-block>
    </tab>
    <tab title="Kotlin" group-key="kotlin">
    <code-block lang="Kotlin">
    dependencies {
        // ... your other dependencies
        implementation("") {
            isTransitive = false
        }
    }
    </code-block>
    </tab>
    </tabs>