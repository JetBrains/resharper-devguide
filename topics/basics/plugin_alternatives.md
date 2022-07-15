[//]: # (title: Alternatives to Implementing a Plugin)

<!-- Copyright 2000-2022 JetBrains s.r.o. and other contributors. Use of this source code is governed by the Apache 2.0 license that can be found in the LICENSE file. -->

<excerpt>Alternative strategies and tools to avoid building a "full" plugin.</excerpt>

In some cases, implementing an actual ReSharper or Rider plugin can be overkill, and using one of the alternative approaches listed below may provide you with the required value in a much shorter time.
If you need a functionality that is specific to your project domain, conventions, or practices, you can avoid all the steps that are required to implement and publish a plugin and provide these features as a part of your project or IDE configuration files.

Before you start to develop a plugin, define your requirements and verify if they can be covered with any of the alternatives described below.
Consider implementing an actual plugin only when the described solutions are insufficient in your case and there is a significant number of developers who can benefit from it.

## Structural Search and Replace (SSR)

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

## Code-Based SSR Patterns

As an alternative to SSR patterns stored in [settings files](https://www.jetbrains.com/help/resharper/Sharing_Configuration_Options.html), you can define search and replace patterns right in your codebase using [`CodeTemplateAttribute`](https://www.jetbrains.com/help/rider/Reference__Code_Annotation_Attributes.html#CodeTemplateAttribute) from [`JetBrains.Annotations`](https://www.nuget.org/packages/JetBrains.Annotations/) NuGet package. 

The main use case for `[CodeTemplate]` is not notify API users about changes in the API. As the API author, you can mark an obsolete type or member with the `[CodeTemplate]` attribute, where you can specify a search pattern to match the old API and a replacement pattern for it. The attribute will act as a custom code inspection with the corresponding quick-fix.

```csharp
// For example, 'IsTrue()' ia a deprecated API, its usages look like: 'MyAssert.IsTrue(args.Length > 0);'
// The new API usages look like: 'MyAssert.That(args.Length > 0, Is.True);'
// The annotation below will highlight old API usages and suggest replacing them with the new API
[CodeTemplate(
  searchTemplate: "$member$($expr$)",
  Message = "The API is deprecated, use 'MyAssert.That' instead",
  ReplaceTemplate = "MyAssert.That($expr$, Is.True)",
  ReplaceMessage = "Convert to 'MyAssert.That'")]
public static void IsTrue(bool condition)
{
  if (!condition)
    throw new Exception("Assertion failed");
}
```

## Source Templates

[Source Templates](https://www.jetbrains.com/help/resharper/Source_Templates.html) are an alternative to configuring [Live Templates](https://www.jetbrains.com/help/resharper/Creating_a_Live_Template.html) or implementing [Postfix Templates](https://www.jetbrains.com/help/resharper/Postfix_Templates.html). Using the [`JetBrains.Annotations`](https://www.nuget.org/packages/JetBrains.Annotations/) NuGet package, they can be defined right inside your solution with the `SourceTemplateAttribute`. Furthermore, there is a `MacroAttribute` that can reference built-in macros (exposed through the Live Template editor). In contrast to Postfix Templates, Source Templates are less powerful, as there are no semantics that can be considered.
