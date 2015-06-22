---
---

# Getting Started

There are two main ways to extend the ReSharper Platform:

* [**Declarative Extensions**](DeclarativeExtensions.md). This is an extension that distributes settings files that can contain [Live Templates](https://www.jetbrains.com/resharper/help/Templates__Index.html) or [Structural Search and Replace](https://www.jetbrains.com/resharper/help/Navigation_and_Search__Structural_Search_and_Replace.html) patterns, or just application settings. This is surprisingly powerful - Live Templates are a great way to quickly create code from small snippets or even create multiple files, and Structural Search and Replace can automatically and declaratively create analyses and warnings, with Alt+Enter Quick-Fixes, without having to write any code.
* [**Compiled Extensions**](CompiledExtensions.md) (aka plugins). Plugins are .net assemblies that are loaded directly into the ReSharper process and have full access to ReSharper's API, integrating with existing ReSharper features, or providing new features.

Plugins can be created simply by adding the ReSharper SDK NuGet packages to a .net assembly project, but it is strongly recommended to [setup your environment](Tools.md) in order to get a better debugging experience. It is also worth looking at the [structure of the NuGet packages](../Plugins/SDK.md), the [samples](../Plugins/SDK.md), and [how to setup a project](../Plugins/ProjectSetup.md), for [debugging](../Plugins/Debugging.md), local deployment and [testing](../Plugins/Testing.md).

Declarative extensions are simpler and quicker to create than plugins, and can be created with an existing install of ReSharper.

Once an extension has been built, it must be [packaged and deployed](../Extensions/Packaging.md). ReSharper extensions are based on the NuGet infrastructure (although it must be understood that ReSharper extensions are not packaged the same as NuGet packages, and are not intended to be used interchangeably). 

