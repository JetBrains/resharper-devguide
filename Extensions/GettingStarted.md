---
redirect_from:
  - /Intro/GettingStarted.html
---

# Getting Started

There are two main ways to extend the ReSharper Platform:

* [**Declarative Extensions**](DeclarativeExtensions.md). This is an extension that distributes settings files that can contain [Live Templates](https://www.jetbrains.com/resharper/help/Templates__Index.html) or [Structural Search and Replace](https://www.jetbrains.com/resharper/help/Navigation_and_Search__Structural_Search_and_Replace.html) patterns, or just application settings. This is surprisingly powerful - Live Templates are a great way to quickly create code from small snippets or even create multiple files, and Structural Search and Replace can automatically and declaratively create analyses and warnings, with Alt+Enter Quick-Fixes, without having to write any code. An extension can also deliver External Annotations, which provide hints for compiled assemblies in order to improve ReSharper's inspections.
* [**Compiled Extensions**](CompiledExtensions.md) (aka plugins). Plugins are .net assemblies that are loaded directly into the ReSharper process and have full access to ReSharper's API, integrating with existing ReSharper features, or providing new features. The SDK for building plugins is delivered as a set of NuGet packages.

Declarative extensions are simpler and quicker to create than plugins, and can be created with an existing install of ReSharper.

Plugins require more effort to create than declarative extensions, and it is strongly recommended to read the guide sections on how to [set up your environment](Tools.md), [project setup](/Plugins/ProjectSetup.md), [debugging](/Plugins/Debugging.md), [local deployment](/Extensions/Deployment/LocalInstallation.md) and [testing](/Plugins/Testing.md). Plugins can also include settings files and external annotations.

Once an extension has been built, it must be [packaged and deployed](/Extensions/Packaging.md). ReSharper extensions are based on the NuGet infrastructure (although it must be understood that ReSharper extensions are not packaged the same as NuGet packages, and are not intended to be used interchangeably). 

