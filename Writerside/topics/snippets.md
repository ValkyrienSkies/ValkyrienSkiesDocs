---
is-library: true
title: Snippets
---

<snippet id="gradle_add_repository">

Find the `repositories` block, or create one if you don't have it. Add the Valkyrien Skies maven
repository to the `repositories` block. It should look like this:

<tabs group="ktg">
<tab title="Groovy" group-key="groovy">
<code-block lang="Groovy">
repositories {
    // ... your other repositories
    maven {
        url = 'https://maven.valkyrienskies.org'
    }
}
</code-block>
</tab>
<tab title="Kotlin" group-key="kotlin">
<code-block lang="Kotlin">
repositories {
    // ... your other repositories
    maven {
        url = uri("https://maven.valkyrienskies.org")
    }
}
</code-block>
</tab>
</tabs>

</snippet>

<snippet id="var_version">


</snippet>