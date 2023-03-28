[//]: # (title: Registering a Custom Language)

Before a custom language can be recognised and parsed by ReSharper, it must first be registered. There are several classes that need to be registered:

* [Project Model file type](ProjectFileType.md)
* [PSI language definition](PsiLanguageDefinition.md)
* [Project Model language service](ProjectFileLanguageService.md)
* [PSI language service](PsiLanguageService.md)

If the custom language is the primary language of a file, the [file type needs to be registered with the Project Model](ProjectFileType.md). This registration maps file extensions to the file type, and tells ReSharper that a specific file is a "known file". This file type is also used to provide [file type specific components](Registration_PerLanguageComponents.md), such as the project file language service that can be used to retrieve the PSI implementation for that language. The file type is represented by an instance of `ProjectFileType`, specifically a subclass of `KnownProjectFileType`.

A custom language can be the primary language of a file, or it can be secondary, such as C# "islands" in a `.aspx` page, or injected, for example, into a C# string literal that contains a regular expression. If the language is secondary or injected, the custom language doesn't need a project file type.

[Registering a PSI language definition](PsiLanguageDefinition.md) tells the PSI about the language. Every node in the PSI abstract syntax tree returns this language type from `ITreeNode.Language`. The language type is represented by a derived instance of `PsiLanguageType`, specifically a subclass of `KnownLanguage`. The language type is also used to provide [language specific components](Registration_PerLanguageComponents.md), such as the language service that provides access to the lexer and parser.

If a file doesn't have a Project Model file type, or a PSI language mapping, it will be represented by `UnknownProjectFileType` and `UnknownLanguage`, respectively.

Once the file type and language definitions are registered, a [Project Model language service](ProjectFileLanguageService.md) is required to act as a bridge between the Project Model and the PSI, to allow mapping between a file extension and the project file type. Finally, a [PSI language service](PsiLanguageService.md) provides access to the lexer and parser, so that the PSI tree can be built.
