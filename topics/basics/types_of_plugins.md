[//]: # (title: Plugin Types)

<!-- Copyright 2000-2022 JetBrains s.r.o. and contributors. Use of this source code is governed by the Apache 2.0 license. -->

Products based on the ReSharper Platform can be modified and adjusted for custom purposes by adding plugins.
All downloadable plugins are available from the [JetBrains Marketplace](https://plugins.jetbrains.com/).

The most common types of plugins include:

* Library support
* Tool integration
* Inspections and quick-fixes
* Context Actions

> In some cases, implementing an actual ReSharper Platform plugin might not be necessary, as [alternative solutions](plugin_alternatives.md) exist.
>
{type="tip"}

## Library Support

Library support provides improved code insights, which usually cannot be expressed through the compiler and its type-system, that includes:

* Code completion
* Code inspections and quick-fixes
* Reference providers

## Tool Integration

Tool integration makes it possible to manipulate third-party tools and components directly from the IDE without switching contexts, that implies:

* Implementation of additional actions
* Related UI components
* Access to external resources

## Custom Language Support

Custom language support provides basic functionality for working with a particular programming language, that includes:

* File type recognition
* Lexical analysis
* Syntax highlighting
* Formatting
* Code completion
* Code inspections and quick-fixes
* Context Actions
* Navigation

Plugins can also augment existing (bundled) custom languages, e.g., by providing additional inspections, quick-fixes, or any other features.
