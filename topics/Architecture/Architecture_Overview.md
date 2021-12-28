[//]: # (title: Architectural Overview)

In any given ReSharper plugin, you are likely to be interacting with many different subsystems of the ReSharper product. In this part of the guide, we are going to take a look at a bird's eye view of ReSharper.

At the top level, we can isolate the following aspects of ReSharper:



## Platform

The Platform part of ReSharper typically relates to features which interact directly with Visual Studio. As a result, plugin developers are unlikely to work with Platform components directly. However, it's important to have a general idea of some of the platform components in order to be able to understand and investigate, if necessary, the ways in which interactions happen. Here are a few Platform elements that are worth knowing about:

* *ActionManagement* - this assembly concerns itself with managing actions, i.e., executable pieces of code that appear in Visual Studio's Command system. See [Actions and Menu Items](Actions.md) for more information on creating actions.
* *Annotations* - relates to the Annotations subsystem. The idea of annotations is being able to use attributes to give ReSharper hints as to how the code behaves. Frequently encountered attributes such as `[NotNull]` are examples of annotations.
* *ComponentModel* - relates to ReSharper's component model. Understanding of the component model is critical to be able to use various ReSharper's subsystems. See [Component Model](Platform_ComponentModel.md) for more details.
* *DocumentManager* and *DocumentModel* - contains mechanisms for managing documents. Encapsulates important concepts (e.g., `RangeMarker`) used for correctly modifying documents.
* *ProjectModel* - used for interacting with Visual Studio's project model.
* *TextControl* - used for interacting with Visual Studio's text editor control.
* *Util* - contains a large number of important ReSharper utility entities, including specialized collections, DataFlow classes (Lifetime) and many others.

Please note that Platform elements typically live in namespaces unrelated to ReSharper itself. The reason for this is that the Platform components are used not only in ReSharper, but also in other products such as dotTrace, dotCover and dotPeek.

## PSI

The PSI or the Program Structure Interface, is the part of ReSharper responsible for lexing and parsing the languages that ReSharper supports. It is also responsible for resolving types and such practicalities as, e.g., overload resolution, type inference, etc.

The PSI exists in several assemblies: the general Psi assembly, language-specific assemblies (e.g., Psi.CSharp), platform-specific assemblies (e.g., Psi.Web) as well as specialized multi-language assemblies such as Psi.Html.JavaScript. The latter assemblies are used specifically for situations where files containing several languages are parsed. Such files (e.g., `.cshtml` files of Razor view) are typically found in web development projects.

Central to the PSI is the notion of a Language Service, a component which actually adds ReSharper support for a particular language. The ReSharper PSI assemblies actually contain language services for all languages that ReSharper supports as well as the infrastructure which allows plugin developers to develop new languages.

### PSI Services

Psi Services contain various services built on top of the information provided by PSI itself. An example of such a service would be support for Structural Search. You are unlikely to ever have to use these assemblies directly.

## Features

In ReSharper's terminology, a 'feature' is an aspect of functionality that is provided by ReSharper at the outer, user-visible level. This includes things like code completion, navigation, code cleanup, and many more.

Please note that in developing your own features, you are more likely to be interacting with feature services than with actual features, although feature assemblies are good for learning how to implement certain things.

## Feature Services

As the name suggests, ReSharper's Feature Services contain a large number of services that support ReSharper's features. For example, the component that provides the Code Inspection Wiki data at start-up time is a feature service. Another example would be the various data provider and builder classes that are used in bulb items that ReSharper presents. Support for the [Generate Menu](GenerateMenu.md) and many other features is also contained here.

Please note that feature services do not contain features themselves. Quite often they contain components which are used by more than one feature.

### Daemon

The Daemon assemblies are the background-running tasks that analyze source and binary code, react to changes in the solution or the environment, and allow the possibility of highlighting code based on said analyses.

If you require any sort of background analysis on existing code, it's likely you need a daemon. To learn how to write one, see [Daemons and Daemon Stages](Daemons.md).

### Intentions

Intentions are concerned primarily with [Quick-Fixes and Context Actions](Actions.md) - those bulb-bearing popup menus that show up in code either in relation to daemon highlightings or depending on the current context (i.e., the location of the caret).

All the important base classes for quick-fixes and context actions live here. In addition, these assemblies actually contain the quick-fixes and context actions that are shipped with ReSharper.

### Live Templates

This is where support for Live Templates, ReSharper's snippet-based code generation mechanism, is contained. It is probably of little interest to plugin developers save for those wishing to write [Live Template Macros](LiveTemplates.md).

### Refactorings

ReSharper's support for [Refactoring](Refactoring.md) is contained here. These assemblies contain the necessary base classes in order to create and execute your own refactoring workflows.