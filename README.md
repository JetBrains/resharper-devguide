#ReSharper DevGuide

Welcome to the developer guide for the ReSharper Platform. This is the primary source of documentation for developing extensions for the ReSharper Platform that ships as part of ReSharper 9.0.

To view the guide, please visit: http://jetbrains.github.io/resharper-devguide/index.html

The guide is written in Markdown and built by [GitBook](https://www.gitbook.io). It is Open Source and [hosted at GitHub](https://github.com/JetBrains/resharper-devguide). Please feel free to [raise issues](https://github.com/JetBrains/resharper-devguide/issues) or create pull requests to address incorrect or missing content.

The guide is very much a work in progress (the ReSharper Platform has a very large feature set and API surface), and is by no means a complete reference of the API. The table of contents provides a non-exhaustive list of available features, and shows missing content by greying out the page. Issues are a great way of prioritising what documentation needs to be addressed first. Please let us know what we should focus on!

## What is the ReSharper Platform?

The ReSharper Platform is an Open API for building smart IDE features and developer tools. The platform has a deep understanding of your solution. By parsing the files in your projects, ReSharper can build and cache a syntactic and semantic model of your code, which can be used to perform analysis, showing highlights inline in the IDE as you type. The syntax trees can be manipulated in memory to provide safe refactoring, and a powerful cross-language reference system allows for very rich navigation, search and usage analysis.

The ReSharper product is the main client and extension to the ReSharper Platform. The base platform provides services such as Visual Studio integration, project model abstractions and the infrastructure to work with abstract syntax trees and semantic models. The ReSharper product adds many user facing features on top of this, including different language support, from C# to JavaScript to HTML and CSS, to rich refactoring and unit testing support.

The ReSharper Platform is also very extensible. The whole API is public, and is written with the Open/Closed Principle very much in mind. Both the Platform and the ReSharper product can be extended, to provide new analyses, warnings and fixes, support for new unit test frameworks, and new end-user features, or by supporting new languages, and creating new products and feature sets.

The Platform integrates with Visual Studio, hosting products such as ReSharper, dotTrace and dotCover. But it can also be run standalone, hosting products with their own UI, such as dotPeek, dotMemory and the ReSharper command line tools.

## About this guide

The guide is split into several parts:

* **Part I - Extending the ReSharper Platform**

    Details how the ReSharper Platform can be extended, both declaratively, with Live Templates, annotations and patterns and programmatically, via plugins. It explains how to get started with extensions, setting up the SDK and so on.

* [**Part II - Architectural Overview**](Architecture/Overview.md)

    Provides an introduction and overview of the architecture of the ReSharper Platform, how it is split into several layers - Base Platform, Project Model, Features, custom Products and support for custom Languages. It also introduces the Program Structure Index (PSI) which provides ReSharper with syntactic and semantic models of many different languages.

* **Part III - Platform**

    Describes the base Platform layer of the architecture, that provides many utilities and abstractions, including UI, Visual Studio integration, settings and application composition with the Component Model.

* **Part IV - Project Model**

    Documents the Project Model, which represents the solution and products currently open, abstracting away the differences between target types (console, class library, website, etc.) and also language (C#, VB, JavaScript, etc.)

* **Part V - PSI**

    Describes and provides reference for consuming and manipulating the parsers, abstract syntax trees and semantic views of the file formats that ReSharper understands, such as C#, HTML, CSS, Razor, XML, etc. Documents the powerful reference system which allows for cross-language navigation and refactoring. Also looks at code generation, caching, and control flow.

* **Part VI - Features**

    This section documents the integration points for features built on top of the PSI, from analysis and quick-fixes, to navigation, code completion refactoring and more. Extensions can implement many of these integration points to add new analyses and quick fixes and much more.

* **Part VII - Products**

    The ReSharper Platform hosts multiple products, from ReSharper to dotPeek, dotTrace and dotCover. This section looks at how to create a custom product that lives in the ReSharper Platform. It also takes a look at integration points in the other .net JetBrains products.

* **Part VIII - Custom Languages**

    This section documents how to add support for a language that the ReSharper Platform doesn't know about, creating parsers, abstract syntax trees, semantic views and all of the features built on top.

## Previous versions

This guide is aimed at the latest version of the ReSharper Platform, which is currently shipping with ReSharper 9.0. [Previous versions of the developer guide](https://confluence.jetbrains.com/display/NETCOM/ReSharper+Plugin+Development) are also available, if you wish to build extensions for ReSharper 8.x and earlier.

This guide is aimed at ReSharper 9.0, but many of the features and ideas also apply to ReSharper 8.x and earlier.

