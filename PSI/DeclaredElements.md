---
title: Declared Elements
---

Declared elements are the backbone of ReSharper's semantic model, and provide a language-agnostic entry point into the semantic model of the codebase. While an AST provides a high-fidelity model of a code file, it is a representation only of the syntax - the typed text. It does little to help with the semantic view of the underlying code. For example, a definition of a C# class may be split across multiple files by using the `partial` keyword. Retrieving all type members from the AST now requires processing multiple ASTs, and the existing AST gives no indication of the location of the other part of the file. Similarly, if a C# class derives from the class `Foo`, the AST gives no indication which class `Foo` - what namespace does it belong to? A semantic model provides all of this information, for example, applying the type resolution rules to the `using` statements at the top of the file.

At its simplest level, a declared element is "something that has declarations". This can be a CLR class declaration, or a method declaration, or something not even related to code - HTML elements, CSS classes and even colours and file system paths.

All declared elements implement interfaces that derive from the language agnostic `IDeclaredElement` interface. For example, CLR types are represented with the `ITypeElement` interface, which provides methods and properties for getting at the declared elements for the target type's methods, properties, constructors and so on. Since a type's members are also represented as declared elements, it is (almost) possible to use declared elements to model an entire codebase. There are however, some scenarios that declared elements can't model, and these are documented in the [Type System](TypeSystem.md) section.

The `IDeclaredElement` interface provides the `GetDeclarations` method that returns one or more `IDeclaration` instances. The `IDeclaration` interface derives from `ITreeNode` and is defined by the PSI for multiple languages to use for the declarations found in source code. For example, C#'s `IClassDeclaration` tree node inherits from `IDeclaration`, allowing a language agnostic way of working with tree nodes that are also declarations. This links a declared element to its declaration in source, allowing code that has a method declaration to walk the AST of that declaration to look at the method body.

Similarly, the `IDeclaration` interface provides a `DeclaredElement` property, which allows code to get to a declared element, given a declaration node in the AST.

