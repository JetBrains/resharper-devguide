---
---

# About this guide

The guide is split into several parts:

* [**Part I - Extending the ReSharper Platform**](/Intro/GettingStarted.md)

    An overview of the ways in which the ReSharper Platform can be extended, both declaratively, with Live Templates, annotations and patterns and programmatically, via plugins. It explains how to get started with extensions, setting up the SDK and so on. It also includes a migration guide to help migrate plugins and extensions from previous versions of ReSharper.

* [**Part II - Architectural Overview**](../Architecture/Overview.md)

    Provides an introduction to and overview of the architecture of the ReSharper Platform, looking at how it is split into several layers - Base Platform, Project Model, Features, custom Products and support for custom Languages. It also introduces the Program Structure Index (PSI) which provides ReSharper with syntactic and semantic models of many different languages.

* **Part III - Platform**

    Describes the base Platform layer of the architecture, that provides many utilities and abstractions, including UI, Visual Studio integration, settings and application composition with the Component Model.

* **Part IV - Project Model**

    Documents the Project Model, which represents the solution and products currently open, abstracting away the differences between target types (console, class library, website, etc.) and also language (C#, VB, JavaScript, etc.)

* [**Part V - Program Structure Index (PSI)**](../Architecture/PSI.md)

    Describes and provides reference for consuming and manipulating the parsers, abstract syntax trees and semantic views of the file formats that ReSharper understands, such as C#, HTML, CSS, Razor, XML, etc. Documents the powerful reference system which allows for cross-language navigation and refactoring. Also looks at code generation, caching, and control flow.

* **Part VI - Features**

    This section documents the integration points for features built on top of the PSI, from analysis and quick-fixes, to navigation, code completion refactoring and more. Extensions can implement many of these integration points to add new analyses and quick fixes and much more.

* **Part VII - Products**

    The ReSharper Platform hosts multiple products, from ReSharper to dotPeek, dotTrace and dotCover. This section looks at how to create a custom product that lives in the ReSharper Platform. It also takes a look at integration points in the other .net JetBrains products.

* [**Part VIII - Custom Languages**](../CustomLanguages/Overview.md)

    This section documents how to add support for a language that the ReSharper Platform doesn't know about, creating parsers, abstract syntax trees, semantic views and all of the features built on top.


