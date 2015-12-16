---
---

# Per-Language Components

ReSharper is designed to work with multiple languages and file types. It provides features that are cross language, but that require language specific implementations. For example, formatting is a feature that works across all languages, but each language must provide its own formatting rules.

If these per-language implementations were implemented as shell or solution components, then any consumers would need to retrieve all implementations and filter them appropriately, either by a method on the implemented interface, or some other metadata, such as attributes.

Per-language components provides this infrastructure, by creating extra component containers, for specific languages. This means there is a component container for all C# language components, or JavaScript components, and so on. These containers chain to the shell component container, so language specific components can consume any other per-language components of the same language, or any shell components. This also means that per-language components have the same lifetime as shell components.

There are two types of language specific components - project file type specific, and PSI language specific, and each type has their own per-language container.

## File type specific components

File type specific components are specific to a particular [Project Model file type](ProjectFileType.md). They are defined as classes that are marked with the `[ProjectFileType(...)]` attribute. The attribute will take the type of the Project Model's `ProjectFileType` for which this component is intended. For example, a component that needs to be C# specific would be marked with `[ProjectFileType(typeof(CSharpProjectFileType))]`.

These file type specific components can only be retrieved using the methods on the `IProjectFileTypeServices` shell component, or by injecting into other components specific to the same file type. They can also consume shell components, and the lifetime is the same as shell components.

Since project file types are hierarchical, a component can be marked with a base class project file type, and still be available when requested from a more specific file type. For example, a component marked with `[ProjectFileType(typeof(HtmlProjectFileType))]` could be returned for a consumer that wants a service for `AspProjectFileType`, since `.aspx` files "inherit" from HTML files. Of course, if there is an implementation of the requested component that is marked for `AspProjectFileType`, the more specific file type implementation is returned.

## PSI language specific components

A PSI language specific component is very similar to a file type component, except the component is not specific to a Project Model file type, but to a [PSI language definition](PsiLanguageDefinition.md). These components are marked with the `[Language(...)]` attribute, where the parameter is the type of the language definition the component is specific to. For example, `[Language(typeof(CSharpLanguage))]`.

To get a language specific component, the consumer should use the `ILanguageManager` shell component, or inject it into the constructor of another component that is also specific to the same language. Just like file type specific components, language specific components can consume shell components, and this means language specific components have the same lifecycle as a shell component.

See the section on [per-language components](/PSI/PerLanguageComponents.md) in the PSI guide for more details.
