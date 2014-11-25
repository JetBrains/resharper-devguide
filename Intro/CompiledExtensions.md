# Compiled Extensions (Plugins)

ReSharper supports two types of extensions - compiled and [declarative](DeclarativeExtensions.md). A declarative extension is an extension that contains External Annotations, or features that can be stored in settings files, such as Live Templates or Structural Search and Replace patterns.

A compiled extension, known more simply as a plugin, is a .net assembly that is loaded by the ReSharper Platform, and has full access to the API. ReSharper's doesn't have a "plugin API", preferring to make the whole API public and allow plugins to make full use of the platform. This gives the plugin author great power - plugins have full access to the same API as ReSharper's native functionality.

The downside to a fully open API is that [each release of ReSharper introduces breaking changes](PlatformVersioning.md). A "plugin API" that was a controlled subset of the full API could avoid some, but not all, of these changes. The biggest issue is that changes to supported languages requires changes to the structure of the abstract syntax trees - breaking changes are essentially guaranteed when updating language support. This negates any benefit of having a stable, limited, plugin API, and ReSharper allows full access to the API - if it's public, you can use it!

## What can a plugin do?

The ReSharper Platform is massively extensible. It is designed as a very loosely coupled, component based application. The whole application is made up of lots of components that are wired together at runtime by the [Component Model](../Platform/ComponentModel.md), with dependencies being automatically passed as constructor arguments. This allows components to rely on interfaces, or collections of interfaces without needing to know anything about the implementation. Any component that takes a collection of interfaces is an extension point for a plugin - simply implement the interface, and the Component Model will ensure it's passed to the required component. Essentially, ReSharper is designed around the [Open/Closed Principle](http://en.wikipedia.org/wiki/Open/closed_principle).

There are many, many ways of extending ReSharper, from modifying code completion lists, to formatting code, augmenting navigation, analysing code, Quick-Fixes and Context Actions, even providing support for unit test frameworks. The [Key Topics](KeyTopics.md) page provides a list of common extension points, with links to documentation.

But that list is by no means an exhaustive list of extension points! The component based nature of the application means that there are lots more - the fun part is finding them! This guide aims to cover the majority of the functionality in the ReSharper Platform, but can't list all extension points. dotPeek is a great way to find more, looking at existing implementations, and looking for components that take collections of interfaces.
