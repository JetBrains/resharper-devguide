---
title: Injected PSI
---

ReSharper 7 introduces a new lightweight mechanism for ‘slicing’ a single physical code file into multiple PSI files corresponding to the languages that are used within that file. This mechanism is called _Injected PSI_. An example of an injected PSI file is a JSON file in a property of an HTML control in a JavaScript Metro app. Since the injected PSI mechanism is still being introduced, other pertinent areas (e.g., JS and CSS in HTML) currently do not use this mechanism.

## PSI Types

As it stands, there are two types of PSI files available right now:

* Non-injected PSI files - these are primary PSI files and PSI files implemented by the old mechanisms. One language may only have one non-injected PSI file. All C# and VB files are currently non-injected.
* Injected PSI files - these are PSI files for additional languages, implemented by a brand-new mechanism. At the moment, this mechanism is only used in JSON within HTML in JS Metro apps. In future, this mechanism will handle CSS and JS within HTML and would be also used for new features.

Consequently, API calls to `GetPsiFile()` need to be replaced by corresponding calls that have the extra range parameters. Typically, acquisition of a PSI file requires an additional parameter containing a `DocumentOffset/DocumentRange`. (Alternatively, one can iterate all PSI files, but this may come with a performance cost.) For example:

```csharp
var range = new DocumentRange(textControl.Document, textControl.Caret.Offset());
var htmlFile = (IHtmlFile)projectFile.GetPsiFile<HtmlLanguage>(range);
```

Alternatively, for primary PSI files you can replace these calls to `GetNonInjectedPsiFile()` (this method is contained in the `PsiManagerExtensions` class). You can also use this method for all C# and VB files, even for C# and VB in ASP.NET and Razor, although this is not recommended. For example:

```csharp
var file = process.SourceFile.GetNonInjectedPsiFile<CSharpLanguage>();
```
