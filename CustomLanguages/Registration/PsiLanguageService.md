---
title: PSI Language Service
---

Each custom language must register a PSI language service. This is a [per-language component](PerLanguageComponents.md#psi-language-specific-components) that derives from the `LanguageService` abstract class. It provides the entry points for lexing, parsing and building the PSI tree of a custom language.

As it is the main entry point for functionality for a custom language, it is a rather large base class. Some of the type members have implementations, while others are virtual or abstract. The class looks like this (with implementations elided):

```csharp
public abstract class LanguageService
{
  public abstract ILanguageCacheProvider CacheProvider { get; }
  public abstract ILexerFactory GetPrimaryLexerFactory();
  public abstract ILexer CreateFilteringLexer(ILexer lexer);
  public virtual bool IsValidName(DeclaredElementType elementType, string name);
  public ILazyCachingLexer CreateCachingLexer(IBuffer buffer);
  public virtual IEnumerable<string> EnumerateParserCapabilities();
  public abstract IParser CreateParser(ILexer lexer, IPsiModule module, IPsiSourceFile sourceFile);
  public IFile ParseFile([NotNull] ILexer lexer, [NotNull] IPsiSourceFile sourceFile);
  public abstract bool IsCaseSensitive { get; }
  public abstract bool SupportTypeMemberCache { get; }
  public virtual ITreeNode ParseUsingCapability(string text, string capability, IPsiModule psiModule);
  public virtual PreProcessingDirectivesInFile GetUsedConditionalSymbols (IPsiSourceFile sourceFile);
  public virtual ICodeFormatter CodeFormatter { get; }
  public virtual void OptimizeImportsAndRefs(IFile file, IRangeMarker rangeMarker, bool optimizeUsings, bool shortenReferences, IProgressIndicator progressIndicator);
  public virtual IReferenceContextCodec CreateReferenceContextCodec();
  public IConstantValueService ConstantValueService { get { return myConstantValueService; } }
  public abstract ITypePresenter TypePresenter { get; }
  public virtual IDeclaredElementPresenter DeclaredElementPresenter { get; }
  public virtual bool IsTypeMemberVisible (ITypeMember member);
  public ReferenceAccessType GetReferenceAccessType(IReference reference);
  public virtual ReferenceAccessType GetReferenceAccessType(IDeclaredElement target, IReference reference);
  public virtual IDeclaredElementPointer<T> CreateElementPointer<T>(T declaredElement);
  public virtual ITreeNodePointer<T> CreateTreeElementPointer<T>(T node);
  public virtual IReferencePointer CreateReferencePointer(IReference reference);
  public virtual bool CanContainCachableDeclarations([NotNull] ITreeNode node);
  public virtual bool ParticipatesInClrCaches { get; }
}
```

## Lexing

Lexing is the process of reading a text buffer and producing a stream of tokens, identifying elements within a file, e.g. identifier, keyword, whitespace, comment, etc. The `LanguageService` provides several methods for creating and working with lexers for the custom language.

* **`GetPrimaryLexerFactory`** (abstract) - returns an instance of `ILexerFactory` that can be used to create an `ILexer` for the language. Typically this class is very simple, and will just create an instance of a class that implements `ILexer`.
* **`CreateFilteringLexer`** (abstract) - creates and returns an `ILexer` implementation that takes a given `ILexer` and filters out unnecessary nodes, essentially whitespace and comments. This is used in several places where the code has no interest in whitespace or comments. The typical implementation will derive from the `FilteringLexer` abstract class, and implement the abstract `Skip` method. This class will wrap the original passed in `ILexer`, and call `Skip` to see if a given token type should be skipped.
* **`CreateCachingLexer`** - creates a lexer that operates over a cache of tokens, exposed in the returned `ILazyCachingLexer.TokenBuffer`. This cache can be used to do incremental lexing of a file when only a portion changes, and if the language service implements a cache provider, can also be persisted to ReSharper's caches, to speed up parsing files when opening. The `LanguageService` class provides a default implementation of this method that uses the lexer returned from `GetPrimaryLexerFactory`.

## Parsing

The parser uses the lexer's stream of tokens to produce a concrete syntax tree, or parse tree (aka PSI tree), that represents the structure of the file.

* **`CreateParser`** (abstract) - create an instance of `IParser`, given an `ILexer`, an `IPsiSourceFile` and an `IPsiModule`. The language service should return an `IParser` that can parse the custom language.
* **`ParseFile`** - uses `CreateParser` to parse a file and create an `IFile` instance that represents the root of the PSI tree.
* **`ParseUsingCapability`** - parses some text and return an `ITreeNode`, passing in a string value to denote a language specific "capability". This allows modifying how the text is parsed, without the need to add extra overloads to the language service. For example, the C# parser allows parsing the text as a file, member declaration, statement or expression. The default implementation parses the text as a file, by calling `IParser.ParseFile`.
* **`EnumerateParserCapabilities`** - returns a list of string identifiers that list the parser's capabilities. Only used in the internal PSI Viewer Form utility.

## Services

The language service provides an access point to other language specific services:

* **`CacheProvider`** (abstract) - returns an instance of `ILanguageCacheProvider` that can participate in ReSharper's caches, such as caching lexed tokens, text buffers, type member declarations, and so on.
* **`CodeFormatter`** - returns an instance of `ICodeFormatter` that can be used to format the file, or code blocks within a file. This can be a [per-language component](PerLanguageComponents.md) injected into the constructor.
* **`ConstantValueService`** - returns an instance of `IConstantValueService` used when working with constant values. This is injected into the constructor, and should be implemented as a [per-language component](PerLanguageComponents.md). If a language specific implementation isn't found, then the default `ClrConstantValueService` is used instead.
* **`TypePresenter`** (abstract) - return an instance of `ITypePresenter`, used to get presentable names for an `IType`. This can be a [per-language component](PerLanguageComponents.md) injected into the constructor.
* **`DeclaredElementPresenter`** - return an instance of `IDeclaredElementPresenter` which can return language specific text for an `IDeclaredElement`, as well as parameter kind and access rights (`ref`, `out`, and `public`, `private`, etc.).
* **`CreateReferenceContextCodec`** - return an instance of `IReferenceContextCoded`, which is used to rebind references when inserting a new `ITreeNode` tree into an existing file. Typically, this value will either be `null`, or a new instance of `ReferenceContextCodec`.
* **`IsValidName`** - returns `true` if the given name is a valid name for the given `DeclaredElementType`. Used to check for valid names for identifiers, etc.
* **`IsTypeMemberVisible`** - returns `true` if the given type member is visible. For example, C# will return `false` when given an accessor to a property (such as `get_Name`), while the property itself returns `true`.
* **`GetUsedConditionalSymbols`** - returns a list of preprocessor directives used in a given file, to allow invalidating caches when a conditional symbol changes.
* **`GetReferenceAccessType`** - returns the usage of a given reference target, i.e. invocation, read, write, documentation, etc. Used to filter usages in the Find Results window.
* **`CreateElementPointer`**, **`CreateTreeElementPointer`** and **`CreateReferencePointer`** - allows a language to provide a custom implementation of "pointers" to declared elements, tree nodes and references, such that the items can be found again after a file has been edited. If `null` is returned, a default implementation is used instead.
* **`OptimizeImportsAndRefs`** - optimises import statements, such as C#'s `using` directives. *Legacy, only implemented by C# and VB. No longer used*

## Properties

Various properties can control the way the language service works:

* **`IsCaseSensitive`** - boolean value to indicate if the language is case sensitive or not.
* **`SupportTypeMemberCache`** - returns `true` if the language builds a tree where declarations can be cached.
* **`CanContainCachableDeclarations`** - returns a value to indicate if the given tree node can contain cachable declarations. The default result is `true`. ReSharper walks PSI trees looking for specific tree nodes that can be cached, i.e. those that also implement `ICachedTypeMemberDeclaration` or `ICachedDeclaration2`.
* **`ParticipatesInClrCaches`** - boolean value to indicate that this language uses the standard CLR caches. Only used to verify if the language of a CLR based project has source attached. All languages return true, apart from C++, which does not support CLR based output.
