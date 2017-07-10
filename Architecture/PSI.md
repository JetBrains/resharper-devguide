---
title: PSI
---

The PSI is the Program Structure Index. It builds on the Platform and Project Model layers to create a complete syntactic and semantic view of a codebase. It provides a language agnostic framework for generating abstract syntax trees and semantic models. It also provides an implementation for approximately 20 languages, including C#, VB, XML, HTML, CSS, JavaScript and even regular expressions.

The building blocks of the PSI are:

* **Abstract syntax trees**. The PSI will parse and build abstract syntax trees that represent the syntax in a code file. The trees are represented with a language agnostic base interface `ITreeNode`, and the PSI provides language agnostic utility methods to manipulate the tree, which can be used for code generation, deletion or refactoring. The PSI is responsible for keeping the AST in sync with the underlying code file, updating the text when the tree changes, and reparsing the tree when the text file changes. ReSharper supports resilient and incremental parsing, meaning an AST is always generated even when the code is invalid, and only parts of the file that have changed are re-parsed when the code is edited.

   The PSI also supports secondary and injected ASTs. A secondary AST is based on an in-memory generated file, allowing for code that exists as "islands" in another file, such as C# in .aspx files, or JavaScript in a HTML file. An injected AST is an abstract syntax tree generated from the contents of a node in another tree, such as regular expression support in a C# string literal.

* **Declared elements** are the entry point to the semantic model view of a codebase. Anything that has a declaration should have a declared element. The `IDeclaredElement` interface declares a base type for language agnostic declared elements, and provides a cross language means of referring to declared elements. For the CLR, the `ITypeElement` interface provides a full semantic view of a type, including members, base types and links to the underlying AST declarations and constructs. Declared elements are not limited to types, though, as ReSharper has support for CSS classes, HTML elements and even colours and file system paths.

* **References** are a very powerful mechanism that links an AST node to a declared element, e.g. all type usages have a reference back to the declared element of the type. ReSharper makes extensive use of references, for example, Ctrl+Click navigation is simply following a reference, find usages is a reverse lookup, and rename can be implemented by renaming both the target and any incoming references.

    More powerfully, references can be cross language, meaning any AST node can link to any declared element. This allows for functionality such as a XAML file linking to C# code. This also makes cross language navigation, find usages and rename possible.

* **Type systems** are used by language specific declared element interfaces in order to correctly model type usage. Declared elements can model type declarations, and provide a semantic view of the type, but cannot represent all type usages, such as arrays, pointers or generic types. A type system is used to model this information, usually based on a declared element, and any substitutions required to fulfill a generic type.

* **Caching** allows ReSharper to maintain all of the syntactic and semantic information about the codebase. The PSI provides infrastructure for persistent caching per file, to cache key information about a file, or as an arbitrary key value store, in order to cache information that is not file related, for example unit test result data. It also provides infrastructure for in-memory, transient caches that can be invalidated when the codebase changes.

* **Searching**. The PSI provides functionality to search for references, inheritors, implementors and so on.

* **Control flow analysis**. Different languages can provide control flow analysis graphs, which allow ReSharper to perform value analysis, looking for dead code or `null` value tracking. This is augmented by **constant value services** that can understand constant values defined in code, and add these values into the analysis.
