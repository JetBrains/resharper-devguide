[//]: # (title: Required Experience)

<!-- Copyright 2000-2022 JetBrains s.r.o. and contributors. Use of this source code is governed by the Apache 2.0 license. -->

The ReSharper Platform is based on .NET, implemented mostly in [C#](https://docs.microsoft.com/en-us/dotnet/csharp/). In addition, Rider plugins run as a JVM application, mainly implemented in Java and Kotlin.
At this time, it's not possible to develop plugins for the ReSharper Platform in non-.NET languages.
Developing a plugin for the ReSharper Platform requires knowledge and experience with the following technologies and concepts:
- [.NET](https://dotnet.microsoft.com/en-us/), [C#](https://docs.microsoft.com/en-us/dotnet/csharp/), and its standard and 3rd-party libraries (ReSharper and Rider)
- Java/Kotlin, and other IntelliJ-relevant technologies, as described in the [IntelliJ SDK documentation](https://plugins.jetbrains.com/docs/intellij/plugin-required-experience.html) (Rider only)
- Experience with ReSharper Platform-based tools (e.g., [ReSharper](https://www.jetbrains.com/resharper/) or [Rider](https://www.jetbrains.com/rider/))

Please keep in mind that the ReSharper Platform is a large project, and while we are doing our best to cover as many topics as possible, it is not possible to include every feature and use-case in the documentation.
Developing a plugin will sometimes require digging into [sample projects](https://github.com/JetBrains/resharper-rider-plugin/tree/master/samples), analyzing [other open-source plugins](https://jb.gg/ipe), or [getting help](../Intro/getting-help.md) on our tracked platforms.

It's highly recommended to get familiar with the [](explore_api.md) section before you start the plugin implementation.


> In some cases, implementing an actual ReSharper Platform plugin might not be necessary, as [alternative solutions](plugin_alternatives.md) exist.
>
{type="tip"}
