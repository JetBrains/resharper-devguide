---
title: What's New in ReSharper 9.0?
redirect_from:
  - /Intro/WhatsNew.html
---


This document lists the major changes between ReSharper 8 and ReSharper 9, for extension authors. It is not an exhaustive list, but lists the major breaking changes. Other minor changes may exist. If you wish to update this page, please [edit it](https://github.com/JetBrains/resharper-devguide/edit/master/Intro/WhatsNew.md), or [submit an issue](https://github.com/JetBrains/resharper-devguide/issues).

* Table of Contents
{:toc}

## Zones and namespace refactorings

ReSharper 9.0 introduces the ReSharper Platform, which is a common binary distribution of the platform used to build ReSharper, dotCover, dotTrace, dotPeek and other .net based JetBrains tools. Zones are a mechanism to partition components into feature sets that can be enabled or disabled, or required by one product, but not another. This is used to stop usage requirements dictating packaging concerns. That is, if a particular language feature is used by product A but not product B, it can still be implemented in a single assembly targeting that language, rather than having to arbitrarily split the assembly into two, one for each shipping assembly.

Components must now declare the zones they require in order to be activated. If any of the required zones are unavailable (due to product support or configuration), the component will also not be available. For example, if a plugin publishes a component that implements a Quick-Fix, it must declare that it requires Quick-Fixes. If the product doesn't support Quick-Fixes (e.g. dotPeek, or dotTrace), then the component isn't instantiated. This, coupled with the single binary distribution, means that plugins are now applied to *all* products, not just ReSharper. If the dependent feature is available, the plugin will load in that product.

Zones are based on namespaces - a zone marker class declares the required zones for all components in that namespace and below. Due to this, many namespaces in ReSharper have been renamed to better reflect the hierarchical classification zones dictate.

Existing plugin authors will have to update their plugins to work with zones. Firstly, many errors will simply be namespace renames. Using ReSharper to automatically import the types from their new namespaces should fix many of these errors. Secondly, the plugin author should add a zone marker class describing the zones required for the plugins functionality. ReSharper provides an internal analysis to help with this.

See the [section on Zones](/Platform/Zones.md) for more details.

## Extension packages

Extensions are now extensions to the ReSharper Platform, instead of extensions to just ReSharper. This requires some changes to the way extensions are packaged, and deployed. They are still NuGet packages, but the layout of the content has changed. Instead of putting the extensions in a root folder called "ReSharper", plugins, external annotations or settings files should be in a folder called "DotFiles". When installed, extensions are now installed directly to the product folder, which is now in `%LOCALAPPDATA%`, rather than `C:\Program Files`.

See the [section on Deployment](/Extensions/Packaging.md) for more details.

## Removal of plugin metadata attributes

The `[PluginTitle]`, `[PluginVendor]` and `[PluginDescription]` attributes have been deprecated and removed. The Plugins Options page has also been removed, meaning the only place this information is displayed is in the Extension Manager, which uses similar information from the extension's package metadata instead. The attributes should be removed.

The ReSharper Platform has changed the way it works with extensions and plugins. Previously, plugins had special treatment from ReSharper - plugins were loaded separately to product assemblies by the Component Model, and were listed in the Plugins Options page. The ReSharper Platform now treats extensions the same as product assemblies - or rather, products are treated the same as extensions. ReSharper, dotCover and dotPeek, etc. are all extensions to the ReSharper Platform. During installation, extensions (1st and 3rd party) are statically registered and are composed at runtime to create a single application. The Extension Manager retrieves extensions and invokes the installer to combine the new extension into the application.

Both the Plugins Options page and the previous version of the Extension Manager allowed for disabling plugins or extensions. This functionality has been removed, in favour of disabling features at the zone level, providing richer control of what can be disabled - an entire product, or one of several features implemented by an extension, be that a 1st party extension like dotTrace, or a 3rd party extension such as a plugin.

A major difference is that this is an opt-in mechanism. In order to allow for disabling a product or a feature, a plugin must advertise a zone as being a product or feature. This is described in the [Features and Products](/Platform/Zones/FeaturesProducts.md) section of the [Zones](/Platform/Zones.md) guide.

## Actions

The Action subsystem has been rewritten for this release. The most obvious changes are:

* `IActionHandler` is to be replaced by `IExecutableAction`.
* Each action handler requires a unique ID.
* Bulk actions have been rewritten.

## Visual Studio integration

The move to the shared ReSharper Platform means that Products can now be viewed as extensions of the ReSharper Platform. This changes how Visual Studio integration is handled. Previous versions would create a separate Visual Studio extension for each Product, e.g. ReSharper, dotCover and dotTrace would all be installed into Visual Studio separately, as individual extensions.

Starting with ReSharper 9, there is only one Visual Studio extension - the ReSharper Platform. Products are installed into the ReSharper Platform, and the Platform will update its registration with Visual Studio to make sure all new assemblies are registered with Visual Studio.

This process also applies to extensions, which are treated in exactly the same manner as Products - when installed into the ReSharper Platform, the extensions assemblies are statically registered with Visual Studio. There are two implications of this:

1. ReSharper extensions can import and export MEF components.
2. Extension "live loading" is no longer supported and Visual Studio must always be restarted when an extension is installed.
3. Actions implemented by extensions will remember shortcut assignments and customisations. This fixes [RSRP-246461](http://youtrack.jetbrains.com/issue/RSRP-246461) and [RSRP-337038](http://youtrack.jetbrains.com/issue/RSRP-337038).

It is strongly recommended to prefer ReSharper abstractions over MEF interfaces wherever possible, as this allows for extensions to be used outside of Visual Studio.

## Background component instantiation

Starting with ReSharper 9.0, the Component Model will no longer immediately create components during application startup, but will instead defer creation until the application becomes idle. This has a very positive impact on application startup performance.

More specifically, any classes marked with attributes such as `[ShellComponent]` and `[SolutionComponent]` are no longer created immediately, as the parts container is composed (e.g. application startup, or solution load), but are queued up to the current thread's `Dispatcher`, at `Background` priority. The components are still created on the main thread, as in previous versions, and all dependencies are still created first.

One implication of this change is that the application can now be up and running before all of the components in the container have been created. Previous versions would create the whole container before allowing the application to continue running. There may be cases where a component should be created in a more timely manner. For example, the Visual Studio integration needs to be created and hooked up before the application can continue. In this case, the component can require "instant" activation, in which case the component (and all of the components required to satisfy constructor dependencies) are created immediately rather than scheduled:

```csharp
[ShellComponent(Requirement = InstantiationRequirement.Instant)]
public class ImmediateComponent
{
}
```

See the [Component Model section](/Platform/ComponentModel.md) for more details.

Note that this only applies when the ReSharper platform is hosted in Visual Studio. Standalone products will always instantiate components immediately.

## Control Flow interface renaming

The control flow graph interfaces have been renamed to correct spelling:

* `IControlFlowGraf` -> `IControlFlowGraph`
* `IControlFlowRib` -> `IControlFlowEdge`

Also, the language specific interfaces have been renamed:

* `ICSharpControlFlowGraf` -> `ICSharpControlFlowGraph` and `ICSharpControlFlowRib` -> `ICSharpControlFlowEdge`
* `IVBControlFlowGraf` -> `IVBControlFlowGraph` and `IVBControlFlowRib` -> `IVBControlFlowEdge`
* `IJsControlFlowGraf` -> `IJsControlFlowGraph` and `IJavaScriptControlFlowRib` -> `IJsControlFlowEdge`

The `IControlFlowBuilder` and `ControlFlowBuilder` provide language agnostic methods to create a control flow graph.

## Search in options

Options pages are now searchable. To implement in your own options page, either implement `ISearchablePage`, or derive from `SimpleOptionsPage`. Highlighting results is implemented automatically for XAML pages, as long as the page sets the attached property `SearchablePageBehavior.SearchFilter="True"` on the root control of the page, while also setting the `DataContext` of the root control to the implementation of `ISearchablePage`.

## Unit test provider changes

### Unit test element ID changes

The ID for unit test elements is no longer a simple string ID. This change is to support multiple providers in the same project (e.g. mixing nunit and xunit tests), and also to allow for tests with the same fully qualified class or method name in the same solution. This is especially important for Shared Projects.

Instead of the ID being a simple string, it is now a `UnitTestElementId` instance, which is a value type (multiple instances can be considered equal if the fields are all equal). It is comprised of three values - the test provider (e.g. nunit, xunit), a `PersistentProjectId`, which is usually a GUID in the `.csproj` file, and the old string ID of the element. The string ID of the element can be the fully qualified name of the class or method - taking all three values into account allows for uniqueness of IDs.

The persistent project ID can be retrieved by calling `IProject.GetPersistentId()`.

### File and metadata explorers

The `IUnitTestMetadataExplorer`, `IUnitTestFileExplorer` and the `ExploreExternal` and `ExploreSolution` methods on the `IUnitTestProvider` interface have been merged into a single interface, `IUnitTestElementSource`. This should be implemented as a `[SolutionComponent]`. It provides a direct replacement for the above mentioned interfaces and methods, allowing for unit test discovery from PSI syntax trees or assembly IL metadata. The `IUnitTestElementSource.ExploreSolution` method allows for discovery of test elements via other, non-prescribed means, perhaps by examining the solution for non-project files, or looking for project files that don't have a PSI syntax tree.

The implementation of the interface can defer to the existing implementations of `IUnitTestMetadataExplorer` or `IUnitTestFileExplorer`, with minor alterations. The `UnitTestElementConsumer` delegate has been replaced with a more flexible `IUnitTestElementsObserver`, and the `MetadataElementsSource` class provides infrastructure for loading assembly IL metadata and calling an `ExploreAssembly`-like method.

