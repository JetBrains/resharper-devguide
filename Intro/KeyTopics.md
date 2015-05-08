---
---

# Key Topics

The ReSharper Platform is very large, and very capable, and its size can initially be very daunting. This page is intended to list the key topics that a plugin author would be interested in, and provide quick links to the most common extension points.

## Basics

* [Getting Started](GettingStarted.md) with extensions and plugins.
* [Architectural overview](../Architecture/Overview.md) - a brief tour of the different layers of the ReSharper Platform.
* [`Lifetime`](../Platform/Lifetime.md) - a key class used across the whole of the Platform. Provides lifetime management features.
* [Component Model](../Platform/ComponentModel.md) - ReSharper is a highly component based application, importing and exporting services and components. To get the most out of it requires some understanding of how the component model works, and how it relates to lifetime management.
* [Zones](../Platform/Zones.md) - the ReSharper Platform's shared binary distribution raises issues about shipping mechanisms (i.e. how to ship a component that is used by one product configuration but not another). Zones manage this by partitioning components into logical feature sets. Extension providers need to understand how this works in order to correctly import and export components.

Also, there is a [migration guide](../Intro/WhatsNew.md) for existing extension authors.

## The code model

ReSharper's code model is called the PSI - the Program Structure Index. The key topics here are:

* [Overview](../Architecture/PSI.md)
* [Abstract syntax trees](../PSI/SyntaxTrees.md) - files are parsed and abstract syntax trees are generated, as you type.
* [Declared elements](../PSI/DeclaredElements.md) - declared elements are the root interfaces for ReSharper's semantic view of the code.
* [References](../PSI/References.md) - a very powerful mechanism to allow abstract syntax trees to have an outgoing references to a declared element, that is, to link syntax nodes, such as type names, to the semantic item they refer to.
* [Type Systems](../PSI/TypeSystems2.md) - modelling type usage, based on the semantic view of the code given by declared elements.

These are the core building blocks of the PSI, although it has more functionality, including caching, searching and control flow analysis. The PSI section has more details.

## Common extension points

ReSharper is extremely extensible. It is a heavily component based application, and is designed to follow the Open Closed Principle to allow for such extensibility. Most features and services can be extended, but some common extension points are:

* [Daemons and daemon stages](../Features/Analysis/Daemons.md) - code analysis, as you type. Displays "squiggly" warnings in your code.
* [Actions](../Features/Actions.md) - menu items, and toolbar items.
* [Quick-Fixes and Context Actions](../Features/Actions/QuickFixes.md) - extending the Alt+Enter menu.
* [Code Completion](../Features/Completion.md) - extend the code completion menus.
* [Reference Providers](../PSI/References/ReferenceProviders.md) - apply references from an arbitrary abstract syntax tree node to another code element, such as supporting code completion and navigation from string literals to class or property names.
* [Generate Menu](../Features/GenerateMenu.md) - add items to the Generate menu.
* [Navigation](../Features/Navigation.md) - extend navigation.
* [Options Pages](../Features/OptionsPages.md) - add your own options pages.
* [Code Cleanup](../Features/Tools/CodeCleanup.md) - participate in code cleanup.
* [Live Templates](../Features/LiveTemplates.md) - add macros for Live Templates.
* [Refactoring](../Features/Refactoring.md) - add your own refactorings.

