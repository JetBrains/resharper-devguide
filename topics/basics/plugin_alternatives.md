[//]: # (title: Alternatives to Implementing a Plugin)

<!-- Copyright 2000-2022 JetBrains s.r.o. and other contributors. Use of this source code is governed by the Apache 2.0 license that can be found in the LICENSE file. -->

<excerpt>Alternative strategies and tools to avoid building a "full" plugin.</excerpt>

In some cases, implementing an actual ReSharper or Rider plugin can be overkill, and using one of the alternative approaches listed below may provide you with the required value in a much shorter time.
If you need a functionality that is specific to your project domain, conventions, or practices, you can avoid all the steps that are required to implement and publish a plugin and provide these features as a part of your project or IDE configuration files.

Before you start to develop a plugin, define your requirements and verify if they can be covered with any of the alternatives described below.
Consider implementing an actual plugin only when the described solutions are insufficient in your case and there is a significant number of developers who can benefit from it.

## Structural Search and Replace

> As of now, Rider only reads existing SSR patterns but does not provide a UI to add or change existing patterns. Feel free to subscribe to the [tracking issue](https://youtrack.jetbrains.com/issue/RIDER-11489).
>
{type="note"}

The [Structural Search and Replace (SSR)](https://www.jetbrains.com/help/resharper/Navigation_and_Search__Structural_Search_and_Replace.html) functionality allows defining search patterns which are based not only on textual information but also on the structure of the searched code fragments, no matter how it is formatted or commented.
The SSR templates can be used for [creating custom inspections](https://www.jetbrains.com/help/resharper/Code_Inspection__Creating_Custom_Inspections_and_QuickFixes.html), which can be an alternative for [programmatic code inspections](#).
Depending on requirements, an inspection can report an issue for a code fragment matching a given template, but also provide a quick fix replacing the reported fragment with the configured replacement template.
All inspection metadata like name, problem tooltip, and description are configurable.
A single inspection can use multiple search and replacement templates.

Once SSR inspections are created and configured, they can be shared with other team members via [settings layers](https://www.jetbrains.com/help/resharper/Reference__Settings_Layers.html).

SSR inspections can be created only for languages providing SSR support.
To verify if a given language supports SSR, invoke the <menupath>ReSharper | Find | Search with Pattern...</menupath> action, and check if it is present in the <control>Language</control> select list.

## Source Templates

[Source Templates](https://www.jetbrains.com/help/resharper/Source_Templates.html) are an alternative to configuring [Live Templates](https://www.jetbrains.com/help/resharper/Creating_a_Live_Template.html) or implementing [Postfix Templates](https://www.jetbrains.com/help/resharper/Postfix_Templates.html). Using the [`JetBrains.Annotations`](https://www.nuget.org/packages/JetBrains.Annotations/) NuGet package, they can be defined right inside your solution with the `SourceTemplateAttribute`. Furthermore, there is a `MacroAttribute` that can reference built-in macros (exposed through the Live Template editor). In contrast to Postfix Templates, Source Templates are less powerful, as there are no semantics that can be considered.

## Code Templates

As an alternative to SSR patterns stored in [settings files](https://www.jetbrains.com/help/resharper/Sharing_Configuration_Options.html), you can also define Code Templates right in your codebase using the [`JetBrains.Annotations`](https://www.nuget.org/packages/JetBrains.Annotations/) NuGet package. In contrast to SSR patterns, Code Templates are defined as pure strings and therefore, there is no UI guidance when writing them. Though, once the `JetBrains.Annotations` package is referenced, you can take a look at the comprehensive XML documentation for the `CodeTemplateAttribute`.

```csharp
namespace MyNamespace
{
    class Customer
    {
        public List<string> Orders => null;

        [CodeTemplate(
            searchTemplate: "$customer{Expression,'MyNamespace.Customer'}$.GetOrders()",
            Message = "Use property instead of the method call",
            ReplaceMessage = "Replace with property",
            ReplaceTemplate = "$customer$.Orders")]
        public List<string> GetOrders() => Orders;
    }
}
```
