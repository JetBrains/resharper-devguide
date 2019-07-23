---
title: Rider plugins
---


Rider is a cross-platform .NET IDE. While all language-related features discussed in this guide can be run by Rider in a ReSharper background process, the actual IDE frontend is based on the IntelliJ platform. The ReSharper backend is either running on the .NET Framework or Mono, depending on installations and operating system. The frontend, however, is running on the JVM. In order to make those two process talk with each other and let them share data, some kind of protocol is required. For instance, the frontend might need to tell the ReSharper process that we hit <kbd>Tab</kbd> after a `value.`, and that auto-completion items should be calculated. In return, the background process needs to share all the possible items with the frontend, so that the user can choose from a list. This protocol is implemented using [Kotlin](https://kotlinlang.org/).

As an author of a ReSharper extension, we might want to make it available for Rider as well. Depending on the types and extension points we are using, this sometimes doesn't require so much additional work, but only to create a Rider [plugin project](#plugin-project-net) and to follow Rider's [plugin packaging](#plugin-packaging). This is usually true if we:

- Don't use any XAML option pages or tool windows
- Don't use any VisualStudio components

In all other cases, we would need to re-implement missing parts using the [IntelliJ Platform SDK](http://www.jetbrains.org/intellij/sdk/docs/welcome.html), which is structured similar to this guide. For inter-process communication, we can easily [extend the protocol](#protocol-extension) using a Kotlin DSL.

## Plugin project (.NET)

The standard approach of migrating an existing ReSharper plugin to also work for Rider, is to copy the existing CSPROJ file and adapt the package reference from `JetBrains.ReSharper.SDK` to `JetBrains.Rider.SDK`.

> **NOTE:** It is strongly recommended to use [SDK-based project files](https://docs.microsoft.com/en-us/visualstudio/msbuild/how-to-use-project-sdk) to decrease friction.


We could name these two project files `MyPlugin.RS.csproj` and `MyPlugin.RD.csproj`, which is great for code sharing. Unfortunately, this won't play well when executing MSBuild on them, because both projects would share the same `project.assets.json` file. A simple solution to fix this is to put a custom `Directory.Build.props` file besides the two project files:

```xml
<Project>
  <PropertyGroup>
    <RootNamespace>MyPlugin</RootNamespace>
    <DefaultItemExcludes>$(DefaultItemExcludes);obj\**</DefaultItemExcludes>
    <BaseIntermediateOutputPath>obj\$(MSBuildProjectName)\</BaseIntermediateOutputPath>
    <OutputPath>bin\$(MSBuildProjectName)\$(Configuration)\</OutputPath>
  </PropertyGroup>
</Project>
```

This will provide MSBuild with an individual OBJ and BIN folder both the ReSharper and Rider project. 

## Plugin packaging

Rider plugins are simple ZIP archives containing metadata about the plugin, ReSharper extensions (DLL) and/or IntelliJ extensions (JAR). The content is structured like this:

```
+ plugin-root-folder
    + META-INF
        - plugin.xml
    + lib
        - plugin.jar
    + dotnet
        - plugin.dll
```

Among [other declarations](http://www.jetbrains.org/intellij/sdk/docs/basics/plugin_structure/plugin_configuration_file.html), the `plugin.xml` must provide common metadata about its id, name, version, and frontend dependencies:

```xml
<idea-plugin>
  <depends>com.intellij.modules.rider</depends>
  <idea-version since-build="181.4379" until-build="182.*"/>
  
  <id>com.company.plugin</id>
  <version>1.3.3.7</version>
  <name>MyCompanyPlugin</name>
  <vendor url="https://www.company.com">MyCompany</vendor>
  <description>Some description.</description>
  <change-notes>Some change-notes.</change-notes>
</idea-plugin>
```

For simple plugins that don't contain code that is specific to Rider, we might choose to manually pack the archive. However, this requires us to also manually update the `idea-version`tag according to the *About JetBrains Rider* dialog. Also note that archives created using .NET capabilities (like `System.IO.Compression` or PowerShell's `Compress-Archive`) [might not work](https://youtrack.jetbrains.com/issue/IDEA-180829).

For more complex plugins as well as for better testability, it is recommended to have a dedicated [gradle build](#plugin-project-jvm). This will also automatically take care of updating the `idea-version` tag.

## Plugin project (JVM)

The recommended solution for building Rider frontend plugins is to use the [gradle-intellij-plugin](https://github.com/JetBrains/gradle-intellij-plugin). This usually involves two files `settings.gradle` and `build.gradle`.

Inside `settings.gradle` we define the project name and other global data:

```
rootProject.name = 'rider-MyPlugin'
gradle.ext.resharperPluginProjectName = 'MyPlugin'
``` 

With `build.gradle` we basically use and extend the `intellij` task provided from the gradle plugin. Here is an example:

```
plugins {
    id 'java'
    id 'org.jetbrains.kotlin.jvm' version '1.2.50'
    id 'org.jetbrains.intellij' version '0.3.2'
}

version = version != 'unspecified' ? version : '0.0.0.1'
if (!project.hasProperty('configuration')) ext.configuration = 'Debug'

sourceCompatibility = 1.8
targetCompatibility = 1.8

compileKotlin {
    kotlinOptions { jvmTarget = "1.8" }
}

intellij {
    type = 'RD'
    // Download a version of Rider to compile and run with. Either set `version` to
    // 'LATEST-TRUNK-SNAPSHOT' or 'LATEST-EAP-SNAPSHOT' or a known version.
    // This will download from www.jetbrains.com/intellij-repository/snapshots or
    // www.jetbrains.com/intellij-repository/releases, respectively.
    version = "2018.2-SNAPSHOT"
    // Sources aren't available for Rider
    downloadSources = false
}

prepareSandbox {
    from("$projectDir/../$gradle.pluginName/bin/${gradle.pluginName}.RD/$configuration", {
        into "$intellij.pluginName/dotnet"
        include "$gradle.pluginName*"
    })
}
```

For more information about building IntelliJ plugins, please see the [IntelliJ Platform SDK](http://www.jetbrains.org/intellij/sdk/docs/tutorials/build_system.html).

### Protocol extension

Rider has an extensible protocol to allow communication between IntelliJ frontend and ReSharper backend. This requires our plugin to implement a data model that can be used on .NET and JVM side. For our convenience, the IntelliJ gradle plugin allows to generate these implementations from a Kotlin base definition. Here is an example:

```
package model.rider

import com.jetbrains.rider.model.nova.ide.SolutionModel
import com.jetbrains.rd.generator.nova.*
import com.jetbrains.rd.generator.nova.PredefinedType.*

@Suppress("unused")
object MyUnityModel : Ext(SolutionModel.Solution) {

    val MyEnum = enum {
        +"FirstValue"
        +"SecondValue"
    }
    
    val MyStructure = structdef {
        field("projectFile", string)
        field("target", string)
    }

    init {
        property("myString", string)
        property("myBool", bool)
        property("myEnum", MyEnum.nullable)
        
        map("data", string, string)
        
        signal("myStructure", MyStructure)
    }
}
```

To generate the Kotlin and C# model implementation from that, we need to extend our `build.gradle` file. At the start we need to add a dependency for `com.jetbrains.rd:rd-gen`:

```
buildscript {
    repositories {
        maven { url 'https://www.myget.org/F/rd-snapshots/maven/' }
        mavenCentral()
    }

    dependencies { classpath "com.jetbrains.rd:rd-gen:0.1.18" }
}

plugins { /* ... */ }

apply plugin: 'com.jetbrains.rdgen'
```

Then we can define our own `generateModel` task:

```
ext.rdLibDirectory = new File(repoRoot, "rider/build/riderRD-2018.2-SNAPSHOT/lib/rd")

rdgen {
    def modelDir = new File(repoRoot, "rider/protocol/src/main/kotlin/model")
    def csOutput = new File(repoRoot, "resharper/RdMyPluginProtocol")
    def ktOutput = new File(repoRoot, "rider/src/main/kotlin/com/jetbrains/rider/plugins/myplugin/RdMyPluginProtocol")

    verbose = true
    classpath "$rdLibDirectory/rider-model.jar"
    sources "$modelDir/rider"
    hashFolder = 'build/rdgen/rider'
    packages = "model.rider"

    generator {
        language = "kotlin"
        transform = "asis"
        root = "com.jetbrains.rider.model.nova.ide.IdeRoot"
        namespace = "com.jetbrains.rider.model"
        directory = "$ktOutput"
    }

    generator {
        language = "csharp"
        transform = "reversed"
        root = "com.jetbrains.rider.model.nova.ide.IdeRoot"
        namespace = "JetBrains.Rider.Model"
        directory = "$csOutput"
    }
}
```

### Development

Using the [gradle-intellij-plugin](https://github.com/JetBrains/gradle-intellij-plugin), we can easily launch an experimental Rider installation with our plugin installed right away. We can either start the `runIde` task from the Gradle tool window inside IntelliJ IDEA, or execute `gradlew :runIde` from the command-line. 
