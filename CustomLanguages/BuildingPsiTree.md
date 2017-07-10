---
title: Building the PSI Tree
---

One of the main responsibilities of a custom language implementation is to build a PSI tree which represents the syntactic structure of the file, from class declarations, to keywords, to whitespace and comments. This tree is required for just about all other functionality in ReSharper - refactoring, inspections, formatting, etc. all require manipulating or walking a PSI tree.

> **NOTE** The PSI tree is also commonly referred to as an abstract syntax tree (AST), but it is more correct to call it a concrete syntax tree, or parse tree. An [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) represents the syntactic structure of a file without necessarily being tied to the physical representation of the syntax in a file. For example, parentheses used in an expression to denote grouping aren't needed in an AST as the grouping is implied by the tree structure itself. A [concrete syntax tree or parse tree](https://en.wikipedia.org/wiki/Parse_tree) represents the full syntactic structure of the file, including all the syntax within the file.

ReSharper follows the traditional method of building a parse tree, namely, [lexical analysis ("lexing")](https://en.wikipedia.org/wiki/Lexical_analysis) and [parsing](https://en.wikipedia.org/wiki/Parsing). Lexing is the process of converting an input text buffer into a stream of tokens, such as keyword, identifier, whitespace, operator and so on. Parsing then analyses this stream of tokens, looking for known sequences, such as class declaration, assignment operation, method invocation, etc.

The result of parsing is to build a tree structure, where the leaf nodes represent the tokens from lexing (the [terminal symbols](https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols)), and the interior or branch nodes are semantic constructs, that represent the class declaration, etc. ([non-terminals](https://en.wikipedia.org/wiki/Terminal_and_nonterminal_symbols)). It is now possible to walk the tree to find out what makes up a file, what statements and expressions are in a method declaration, and what identifiers and operators are in an expression.

# PSI tree structure

All languages supported by ReSharper implement a tree that uses the same base interfaces. While the different nodes in each tree are implemented by different classes, they all implement the `ITreeNode` interface, with the root node also implementing `IFile`. This allows for common methods for manipulating and walking the tree. As such, building the PSI tree is a similar exercise for all languages, but it also means that all language support must be implemented from scratch to ReSharper's requirements. There is no direct support for third party tooling, however, as long as the tooling can efficiently create the correct data structures, there is no requirement to use the ReSharper provided tooling (care should be taken to avoid allocations as much as possible - files are reparsed very frequently).

To identify the type of a node, the `ITreeNode` interface includes the `NodeType` property, which is also of type `NodeType`. Each language creates classes that derive from `NodeType` that represent the types of each node in the tree. For example, the leaf nodes of the tree, which represent the tokens identified during lexing, all derive from `TokenNodeType`, and the interior nodes derive from `CompositeNodeType`.

## Lexing

When lexing, ReSharper languages produce a stream of tokens, by returning references to singleton instances of `TokenNodeType` derived classes, such as C#'s `ClassKeywordNodeType`. The lexer needs to implement the `ILexer` interface, which exposes the current token type and the start and end offset of the token itself.

The lexer is usually generated from a `.lex` file that describes the rules to create the tokens. The ReSharper SDK ships a tool to convert the `.lex` file into a class that will implement the lexer. This is a partial class, and requires an accompanying boilerplate partial class that will implement `ILexer` based on the generated lexer.

ReSharper also provides support for incremental lexing, which will reuse existing parts of a stream of tokens when only a portion of a file has changed. The results of lexing can also be persisted to ReSharper's disk based caches, to speed up re-opening a file.

## Parsing

The parser uses an `ILexer` to advance through the file, analysing the token types it discovers along the way. It first creates an instance of `IFile`, and then adds children which represent the top level constructs in the file (e.g. a C# file would have `using` statements, namespace declarations or class declarations). Each child would then have further children (e.g. a namespace declaration would have the `namespace` keyword, whitespace, the name identifier of the namespace as well as the namespace body), and so on (class declarations, type members, expressions, statements, etc.).

The parser uses methods on `TreeElementFactory` to create nodes in the tree (ReSharper also, and perhaps confusingly, uses the term "element" when describing node instances in the tree. That is, implementations of `ITreeNode` are referred to as elements, while the interfaces use "node").

When creating an interior node, the parser passes the node type (a singleton instance derived from `CompositeNodeType`) in a call to `CreateCompositeElement`. This in turn calls `CompositeNodeType.Create`, and the derived instance of `CompositeNodeType` creates the `ITreeNode` instance and returns it, ready to be added to the tree. 

To create a leaf node in the tree, the parser uses the `TreeElementFactory.CreateLeafElement`, passing in the token node type from the lexer. This also produces an `ITreeNode`, which is then added to the tree.

If there is a syntax error in the file, the parser adds a special `ITreeNode` instance that implements `IErrorElement`, and tries to recover with the next recognised lexer tokens. This means the whole file is always parsed, and a valid tree is always created, even if it contains errors.

The parser implements the `IParser` interface, and is usually generated from a `.psi` file, which is a proprietary file format the describes the grammar of the language being parsed. The ReSharper SDK ships with a tool to convert the `.psi` file to a generated class that implements these rules. Alternatively, a parser can be hand written, although it is still useful to use a `.psi` file to generate the tree node classes and interfaces.

A hand written parser is useful if the language is extremely simple, or if the language is too complex to be represented in a `.psi` file easily. Another reason for using a hand written parser is if the language is a superset of another language. For example, TypeScript files are a superset of JavaScript files, so can reuse parts of the parser, for statements, method definitions, etc. A hand rolled parser makes it easier to call into parts of another parser.

ReSharper supports incremental reparsing, whereby only the affected sub-tree is replaced, rather than having to reparse the whole file. This requires special block nodes that know how to reparse their own content.
