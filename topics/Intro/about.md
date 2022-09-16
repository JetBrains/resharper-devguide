[//]: # (title: About This Guide)

<!-- Copyright 2000-2022 JetBrains s.r.o. and other contributors. Use of this source code is governed by the Apache 2.0 license that can be found in the LICENSE file. -->

<excerpt>Introduction and summary overview of contents.</excerpt>

This guide is split into several parts, similar to a textbook.
Each one builds on the content of the previous section, but it is not necessary to read the guide in order.
The [Key Topics](key_topics.md) page aims to link to the pages that are necessary to be able to understand the architecture and get started building plugins.

All source links and reference lists target ReSharper Platform %sdk-version%.

> While browsing this guide, you will notice that there are topics that are greyed out.
> Unfortunately, the guide is not complete and contains placeholders for specific topics.
> We are working on increasing the coverage, but if you get stuck due to missing content, please see the [](getting_help.md) section for details on how to get moving again.
>
> The guide is also [Open Source on GitHub](%guide-repo%), and Pull Requests for new content, corrections or updates are always gratefully received.
> Please see the [Contributing](_CONTRIBUTING.md) page for details.
>
{type="note"}

[//]: # (> See also [Glossary]&#40;&#41; for a handy reference of common terms.)
[//]: # (>)
[//]: # ({type="tip"})

#### Part I — Plugins

Describes how to create a plugin that can extend the ReSharper Platform.
Includes details on how to set up the project, update to new versions of the ReSharper Platform, and how to package, deploy, and test your plugins.

#### Part II — Platform

Describes the foundational layer of the architecture, which provides many features and utilities, such as the component model, the user interface, documents and editors, settings, threading, zoning, and background tasks.
The Platform layer mainly comprises the functionality of the ReSharper Platform that does not target language features or parsing.

#### Part III — Project Model

Documents the Project Model, which represents the files and configuration of the currently loaded solution, as well as the build system used to build the project.

#### Part IV — PSI

The Program Structure Interface (PSI) builds the syntactic and semantic models for lots of different file types.
This section describes how to work with the PSI, navigating and manipulating the syntax trees, and also looks at the powerful reference system, which allows a syntax tree node to reference an item in the semantic model.

#### Part V — Features

Describes how to extend and interact with various features that use the PSI layer, such as code completion, navigation, <shortcut>Alt+Enter</shortcut> items, context actions, refactorings, and more.
See also the section on Custom Languages below for language-specific features that are only applicable when adding support for a new language.

#### Part VI — Testing

Describes the available infrastructure for writing automated tests covering the functionality of plugins.

#### Part VII — Custom Languages

This section describes how to add support to the ReSharper Platform for a new language that isn't supported by default, creating parsers, syntactic and semantic models, and all the features that build on top.

#### Part VIII — Product Specific

A lot of the functionalities in the ReSharper Platform are product agnostic.
This section describes product-specific features, such as specific project model differences and how to target them in a plugin.
