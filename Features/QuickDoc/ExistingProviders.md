# Existing QuickDoc providers

<!-- toc -->

## Overview

ReSharper ships with a number of default providers that create QuickDoc documentation for a broad range of scenarios. Some of these are themselves extensible.

### XML Doc Comments

The `QuickDocTypeMemberProvider` and `QuickDocCommentNodeProvider` classes support XML DocComments. 

* **`QuickDocCommentNodeProvider`** uses a language specific implementation of `IXmDocLocator` to find the closest XML Doc Comment block to the text caret offset, and uses this XML block to display documentation. This is not CLR language specific - XML Doc Comments are supported in JavaScript and TypeScript.
* **`QuickDocTypeMemberProvider`** will look to see if the declared element under the text caret implements `ITypeMember` (which implies a CLR element that also implements `IXmlDocIdOwner`). If so, it gets the XML DocComment Id and then retrieves the XML documentation via a call to `IDeclaredElement.GetXMLDoc`. This also handles documentation shipped with compiled code.

`QuickDocTypeMemberProvider` also has a fallback - if there is no XML documentation, it looks for the `System.ComponentModel.DescriptionAttribute` and uses this for the description of the type member.

### Language specific

Some of the default providers are language specific.

* **`CSharpQuickQueryRangeVariableProvider`** handles documentation for a variable created in LINQ `from` queries. It simply displays the range variable name and type, allowing navigation to the documentation of the type, and a "go to" link to navigate to the declaration of the range variable. Without this provider, the variable would not have any documentation.
* **`QuickDocCssDefinitionProvider`** handles documentation for CSS properties and values. Interestingly, it will call `IDeclaredElement.GetXMLDoc()` on the CSS declared element, which creates an XML node with a `member` element and populates it with `name`, `summary`, `description` and `compatibility` content.
* **`QuickDocJsFunctionProvider`** displays documentation for any element that implements `IJavaScriptTypeOwner`. This displays any XML Doc Comment based documentation for the code element.

### Elements without documentation

Not all code elements map directly to declared elements with documentation. The `QuickDocLocalSymbolProvider` creates simple documentation for local symbols that don't have actual documentation, while the `QuickDocCandidatesProvider` tries to map a reference in the code to a declared element that does have documentation

* **`QuickDocLocalSymbolProvider`** provides a simple description for local symbols based on the type of local symbol (variable, parameter, etc.) and the name of the symbol. Provides "go to" navigation to the declaration of the symbol.
* **`QuickDocCandidatesProvider`** uses the reference at the text caret position to get the declared elements to get the documentation. It will use the language specific `ILanguageReferenceSelector` to try and find language specific candidates (e.g. C# will see if the reference is part of an object creation expression and return the declared elements of the constructor and the type being constructed), or it will simply resolve the reference to find the normal declared element. The provider will use these declared elements to find the appropriate documentation.

    Normally, the providers and the presenters are paired - a specific provider will pass data to a matching provider. But the `QuickDocCandidatesProvider` will find a declared element that is then passed to `QuickDocTypeMemberPresenter`, which means it will look for CLR type members that have XML documentation.

### High priority and fallbacks

The default providers also make use of the `QuickDocProviderAttribute.Priority` property. Most of the providers have a priority of `0`, as they don't overlap in the elements they handle, however, a couple do overlap, and make use of the `Priority`.

* **`QuickDocResxProvider`** has a priority of `-10`, which means that it is called before the other default providers. It checks to see if the current declared element is an instance of `IResourceItemDeclaredElement`, such as the `data` tag in a `.resx` file, or it will look for a CLR property or method that belongs to a resource class. It has to be processed first, or the standard XML Doc Comments handling would take precedence.

* **`QuickDocDescriptionProvider`** has a priority of `1`, meaning it's a fallback to the other default providers. If there isn't a more suitable provider, this one will be used to provide a description of the current declared element. It is used as a fallback as it can only return a single string to act as the description, rather than a fuller piece of documentation like that supplied by XML Doc Comments.

    It retrieves the description by calling into `IDeclaredElementDescriptionPresenter`, which defers to an extensible, live collection of `IDeclaredElementDescriptionProvider`. You can extend this collection by implementing `IDeclaredElementDescriptionProvider` and decorating your class with the `DeclaredElementDescriptionProviderAttribute`.
    
    The default description providers support descriptions for XML nodes via XML schema, CSS compatibility information and HTML element descriptions stored in an internal resource file - which can itself be extended by implementing `IHtmlDeclaredElementsProvider`.

## Extending the default providers

If you wish to extend the default providers, for example, if you're implementing new language support, there are several ways to extend the existing providers to get QuickDoc support:

1. If your language supports standard XML Doc Comments in source, implement `IXmlDocLocator` in a class marked with `[Language (typeof(NewLanguage))]`. The default `QuickDocCommentNodeProvider` class will use this interface to find the XML Doc Comment node.
2. If your language is a CLR language and also supports standard XML Doc Comments in compiled form, your PSI tree should implement `ITypeMember`, and therefore `IClrDeclaredElement` and `IXmlDocIdOwner`. The default `QuickDocTypeMemberProvider` class will retrieve the element's XML doc ID from `IXmlDocIdOwner.XMLDocId` and retrieve the XML itself from `IDeclaredElement.GetXMLDoc()`.
3. You can provide a simple, single string description for your `IDeclaredElement` derived types by implementing `IDeclaredElementDescriptionProvider`, which is used by `QuickDocDescriptionProvider`
4. Implement `ILanguageReferenceSelector` and mark your class with `[Language(typeof(NewLanguage))]`. This class will take in an `IReference` and use it to find any relevant declared elements from which to find documentation. For example, the C# implementation looks for object creation expressions and returns candidates for the constructor and the type being used. This interface is used by the `QuickDocCandidatesProvider`.

Of course, if your requirements don't fit any of these scenarios, you can [implement `IQuickDocProvider` and `IQuickDocPresenter` directly](Implementing.md). You should use the helper methods in `XmlDocHtmlUtil` and `XmlDocHtmlPresenter` to build the HTML.

