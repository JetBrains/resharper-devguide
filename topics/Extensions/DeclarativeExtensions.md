[//]: # (title: Declarative Extensions)

The ReSharper Platform supports two types of extensions - [compiled extensions (aka plugins)](CompiledExtensions.md), and declarative extensions. Plugins are code based, .dll assemblies that are loaded into the process and have full access to the API.

Declarative extensions are either settings files or External Annotation files. They do not contain code, but can be very versatile.

## External annotations

ReSharper provides comprehensive analysis based on looking at your project's source code. However, sometimes it can benefit from extra information. If it has more context about a method, it can produce more accurate warnings, such as better `null` analysis, knowing that a variable in a closure is used immediately (the infamous ["access to modified closure"](https://confluence.jetbrains.com/display/ReSharper/Access+to+modified+closure) analysis), or help for `string.Format` style methods. This context is given by [adding annotations to the source code](https://www.jetbrains.com/resharper/help/Code_Analysis__Code_Annotations.html).

However, if the source is unavailable, it is still possible to add annotations, using [External Annotation files](https://www.jetbrains.com/resharper/help/Code_Analysis__External_Annotations.html), which are XML files that provide a mapping between a pre-compiled assembly's type members, and the annotation attributes that should be applied.

This is a very powerful mechanism and can be used to provide more accurate analysis and warnings to your libraries.

## Settings files

Extensions can also include settings files, that get [mounted as new settings layers](https://www.jetbrains.com/resharper/help/Reference__Settings_Layers.html). This is a very simple, but very powerful extension point. It can be used to distribute:

* **Live Templates** - code snippets and templates, targeting any supported language, or providing templates for a particular framework. An extension can also include a plugin that provides a custom [Live Template macro](Macros.md), to allow for even more customisation.
* **Structural Search and Replace patterns** - a declarative pattern to be used for searching for code, or for highlighting a particular pattern as a warning or error. Furthermore, the pattern can integrate with the Alt+Enter menu to optionally allow for a replacement pattern, to rewrite the matched code, using the value of placeholders in the original text. This allows for declaratively creating analyses and Quick Fixes.
* **Arbitrary settings** - set up standard code formatting, naming patterns, change the default values for inspection severities, etc. Any setting in the Options dialog can be overridden in a .dotSettings file.

This last point is worth repeating. The traditional uses for adding a settings file to an extension is to distribute Live Templates or SSR patterns. However, any setting can be applied in a settings file - disagree with the defaults that ReSharper has chosen? Replace them!