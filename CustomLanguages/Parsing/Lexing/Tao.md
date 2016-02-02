---
---

# The Tao of Writing a Lexer

Writing a lexer for a ReSharper custom language is as easy and as hard as writing any other kind of lexer, but the requirements of writing a lexer for IDE support are slightly different to the requirements for building a compiler or other batch processor. The other pages in this section give technical details on how to implement a lexer. This page is intended to provide insight into what needs to be implemented, and describe what is needed for IDE support.

## Tokens and node types

A lexer for ReSharper is just like a lexer for any other purpose. It needs to read in text via a [text buffer](TextBuffers.md) and convert that into a stream of tokens. It also needs to be very memory efficient, as it will be called frequently, as the user types. As such, it should be written with a lexer generator, such as the [CsLex](CsLex.md) tool that is included in the ReSharper SDK, which will generate a very efficient lexer based on lookup tables, that performs little to no memory allocations.

ReSharper puts certain requirements on lexers, such as [implementing the `ILexer` interface](ImplementingLexers.md). But the most important is the type of tokens that it produces.

A lexer for a custom ReSharper language needs to output tokens that derive from `TokenNodeType`. These are singleton instances, exposed as static fields, which makes comparison and lookup easier. Knowing what nodes to create in the hierarchy, and when to have multiple singleton instances of the same node type can initially seem confusing, but the guidelines are fairly straightforward.

The `TokenNodeType` instances serve three purposes - identification (through the singleton instances), categorisation (through the `IsWhitespace`, `IsKeyword`, etc. properties) and tree node creation. Each node type is responsible for creating the element in the parse tree that represents that token.

Node type classes are split into fixed length or "generic" non-fixed length token node types. The hierarchy of node types for a language start with a base token class, which is immediately derived by fixed and generic node types.

Fixed length tokens are the most straightforward - these are the keywords and other similar tokens from a language, such as static symbols and operators (brackets, braces, semicolons, etc.). They are generated from a `tokens.xml` file, by the [TokenGenerator](/CustomLanguages/Parsing/NodeTypes/CreatingNodeTypes.md#generating-token-node-types) SDK tool. Each keyword and fixed token will get its own class, and just one singleton instance.

Generic, non-fixed length tokens are used for tokens such as identifiers. Each identifier can be different, but will be represented by a single instance of the identifier node type. The text of the token is passed to the tree element at creation, so each tree element knows which identifier it represents.

The following base classes should be created:

* `IMyLanguageTokenNodeType` - a marker interface to easily identify all token node types for a custom language. It can of course include any methods or properties that are common to all token node types.
* `MyLanguageTokenNodeType` - a base type for all token node types. Used only as a base class, this can provide a default implementation of the `IsWhitespace`, `IsKeyword`, etc. properties, either by returning `false` to all and allowing derived classes to override, or by comparing `this` to known singleton instances.

The following classes are usually created that derive directly from `MyLanguageTokenNodeType`:

* `WhitespaceNodeType` and `NewLineNodeType` - simple token node types that represent whitespace and newlines. They will create a instances of `WhitespaceElement` and `NewLineElement` tree elements also defined by the language. The `IsWhitespace` property should return `true`. If whitespace is not significant, `IsFiltered` should also return `true`.
* `IdentifierNodeType` - a non-fixed length token node type that the lexer returns to indicate any identifier. The tree element will be created with the text of the token, so each tree element knows what identifier it represents. The `IsIdentifier` property should return `true`.
* `GenericTokenNodeType` - a non-fixed length token node type that represents any other token that isn't better represented by a more specific type. Typically used as a base type for other node types, or as multiple singleton instances to represent nodes that don't need a specific tree element class. For example, some C# pre-processor tokens can be represented as `GenericTokenElement`, because they don't have any semantics other than identification. Since they don't need their own tree element type, they can share the same node type, but use a different singleton instance, for identification. `GenericTokenNodeType` can also be used to create a singleton to represent bad characters, e.g.:

  ```csharp
  public static readonly TokenNodeType BAD_CHARACTER = new GenericTokenNodeType("BAD_CHARACTER", LAST_GENERATED_TOKEN_TYPE_INDEX + 21, "💩");
  ```

The following classes derive from `GenericTokenNodeType`. If `GenericTokenNodeType` isn't defined, these classes would derive from `MyLanguageTokenNodeType`:

* `CommentNodeType` - used to represent comments. The `IsComment` and `IsFiltered` properties should return `true`. This class can sometimes have subclasses that represent the different comment types - multiline, single line and XML doc comments, or they can all be represented with multiple singleton instances of `CommentNodeType`.
* `FixedTokenNodeType` - a base class to represent tokens of fixed length. The `Create` method will create an instance of a new `FixedTokenElement` type, but is likely to be overridden by more specific token node types, for each keyword and known symbolic token (brackets, braces, semicolons, etc).

The [TokenGenerator](/CustomLanguages/Parsing/NodeTypes/CreatingNodeTypes.md#generating-token-node-types) SDK tool should be used to create keywords and other fixed length tokens. The `tokens.xml` file will describe the token representation of the keyword, and the symbols or whatever else to represent the other fixed length tokens. Symbolic tokens will derive from `FixedTokenNodeType`, and keywords from `KeywordTokenNodeType`.

* `KeywordTokenNodeType` - derives from `FixedTokenNodeType`, and returns `true` for `IsKeyword`. Can define its own tree element type, or let `FixedTokenNodeType` create the tree element.
* Each keyword and symbolic token will have its own singleton instance.

## Lightweight processing

The lexer should be fairly dumb, and not do too much processing of the token stream. The lexer needs to be able to handle invalid and incomplete files, and should ideally be an [incremental lexer](ImplementingLexers.md#incremental-lexers). As such, it should maintain little state, and should push validation of the file to the parser. The parser can insert "syntax error" nodes into the tree, which means it can be responsible for ensuring that a construct is valid.

## Requirements for an IDE

Lexing for an IDE can produce a different set of tokens to lexing for a compiler or other batch process. The purpose of a parser for ReSharper is to generate a concrete parse tree, that represents the semantics of the file while also representing the exact contents of the file. As such, lexical constructs that make sense for a compiler can sometimes be considered too high level for an IDE, and should be deconstructed into smaller constructs.

For example, a lexer for the C# compiler might produce a "namespace identifier" token for the fully qualified namespace in `using System.Collection.Generic;`, while ReSharper's lexer would want to produce multiple tokens that represent each segment of the namespace, to enable richer functionality such as navigation or refactoring of just a section of the namespace.

Similarly, a lexer for ReSharper should ensure that every character in a file is part of a token, leaving no gaps in text.

## Handling syntax errors

The rule of thumb is to leave syntax errors to the parser. The lexer should return a valid stream of tokens, even if they don't make sense for the construct being parsed, and the parser can handle inserting "syntax error" nodes into the tree.

However, the lexer should have a fallback rule, which catches any characters that haven't already been matched, and return a `BAD_CHARACTER` token. Typically, this `BAD_CHARACTER` is a singleton instance of `GenericTokenNodeType`.

## Unterminated constructs

Another requirement for working in an IDE is to work with invalid files. This requires handling cases that might not be mentioned in the language grammar, such as unterminated strings or comments. Typically, a lexer would match a string with start and end quotes, and a multiline comment with start and end symbols (`/*` and `*/`), however, if either construct is unterminated, the lexer should handle reaching the end of the file, and still return a valid token, typically the appropriate string or comment token.

Remember that unterminated constructs should only be matched for single tokens - it is not the lexer's responsibility to handle mismatched braces, for example.

## Matching keywords and identifiers

Languages frequently declare keywords that are essentially reserved identifiers. That is, the keyword matches the rule used to describe identifiers - for example, C#'s `return` keyword could be an identifier if it wasn't declared reserved. In order to properly recognise keywords, the lexer could list each keyword explicitly as its own rule, or could use various [utility classes](UtilityClasses.md) to decide if a matched identifier is a reserved keyword or identifier.

## Write tests

The easiest way to make sure the lexer is correct is to [write tests](Testing.md). Include cases that match as well as cases that don't match. Make sure to test for error conditions such as unterminated constructs.
