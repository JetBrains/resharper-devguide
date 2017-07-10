---
title: Token node types
---

Each leaf element in a PSI tree has a node type that is a singleton instance of a class that derives from the `TokenNodeType` base class, and implements the `ITokenNodeType` interface. This singleton instance is the token produced by the lexer.

```csharp
public abstract class TokenNodeType : NodeType, ITokenNodeType
{
  protected TokenNodeType(string s, int index) : base(s, index) { }

  public virtual LeafElementBase Create(String token)
  {
    return Create(new StringBuffer(token), TreeOffset.Zero, new TreeOffset(token.Length));
  }

  public abstract LeafElementBase Create(IBuffer buffer, TreeOffset startOffset, TreeOffset endOffset);

  // ITokenNodeType properties
  public abstract bool IsWhitespace { get; }
  public abstract bool IsComment { get; }
  public abstract bool IsStringLiteral { get; }
  public abstract bool IsConstantLiteral { get; }
  public abstract bool IsIdentifier { get; }
  public abstract bool IsKeyword { get; }
  public abstract string TokenRepresentation { get; }

  public virtual bool IsFiltered { get { return false; } }

  /// <summary>
  /// For use in formatter, to tell if we need to insert space/new line between two token types
  /// </summary>
  public virtual string GetSampleText()
  {
    return TokenRepresentation;
  }

  /// <summary>
  /// Text to present token type to user, for example in error messages like "{0} expected"
  /// </summary>
  public virtual string GetDescription()
  {
    var s = TokenRepresentation;
    if (s.IsNullOrEmpty()) return ToString();
    return s;
  }
}
```

Each language must provide at least one class that derives from `TokenNodeType`. Typically, this will provide default values for the simple abstract properties. These values can be constants that are overridden in further derived classes (that represent whitespace, comments, etc.) or can be implemented directly by comparing `this` to known singleton values for comments, whitespace, etc.

Like `NodeType`, the constructor takes in a string identifier and a unique index for the node type, and passes them to the base class. The identifier is only used internally, for diagnostics and testing, and should be simple enough to identify the node type, e.g. `"NEW_LINE"`, `"COMMENT"`, `"IDENTIFIER"`, etc.

The `TokenRepresentation` property is a value that is usually passed into the constructor of a derived class, and provides a more human readable representation of the token than the identifier value passed to the constructor of `TokenNodeType`. For example, the identifier might be `"ANDAND"`, while the representation would be `"&&"`, or `"RETURN_KEYWORD"` and `"return"`. This is used by the `GetSampleText` and `GetDescription` methods.

The `GetSampleText` method is called by a language's formatter implementation, to see if two token types require whitespace to separate them (by building a string with the two pieces of sample text and attempting to lex it. If it succeeds, no whitespace is necessary). By default, this is the `TokenRepresentation`.

The `GetDescription` method returns a human readable description of the token for use in parse errors. Typically, this is the `TokenRepresentation` abstract method, but if this is `null` or empty, it returns the string identifier passed to the constructor (e.g. `"IDENTIFIER"`, `"NEW_LINE"`).

The class also provides two `Create` methods. One which takes a string, and another that takes a string buffer and a start and end offset. These methods are used by the parser to create leaf elements for the tree - `LeafElementBase` provides an abstract implementation of various parts of `ITreeNode`.

> **NOTE** For the most part, ReSharper does not rely on a specific implementation of `ITreeNode`. However, returning `LeafElementBase` from `TokenNodeType.Create` indicates that a custom language cannot provide its own implementation of `ITreeNode`. The custom language's parser is the only caller for `TokenNodeType.Create`, so it would be possible to call another method and return a different `ITreeNode`. Unfortunately, certain implementation details mean that the `ITreeNode` needs to be implemented with ReSharper's base classes - primarily, when ReSharper calls the language service to create the root `IFile`, it downcasts it to `FileElementBase`, and this requires all child nodes to be derived from `TreeElement`.

## Token node type hierarchies

A custom language must derive from `TokenNodeType` and provide at least this implementation.

Typically, a language will also create further derived classes to represent whitespace, comments, keywords, identifiers, and so on, essentially creating a derived class for each value of the abstract properties of `ITokenNodeType`.

This can be enough for some languages, while others require a more detailed hierarchy. Several languages implemented by ReSharper also create a `FixedTokenNodeType`, which becomes a base class for tokens that have a fixed (and therefore also fixed-length) representation, such as operators or keywords. This is opposed to whitespace, comments and identifiers, which don't have a fixed, or fixed-length, representation. Typically, this `FixedTokenNodeType` does not add any functionality over `TokenNodeType`, but simply acts as a base class. Sometimes it will provide a default implementation of `Create` that returns a generic implementation of `ITreeNode` that will work for all fixed length tokens.

Similarly, the `KeywordTokenNodeType` derives from `FixedTokenNodeType`, and is used for keywords. Again, there is usually no extra functionality, other than to return `true` from `ITokenNodeType.IsKeyword`.

Sometimes, the language will also define a `GenericTokenNodeType`, that is used as a base class for `FixedTokenNodeType`, and to provide a token node type that is neither an identifier, or whitespace, comment, etc. For example, it can be used to create literals, e.g. `new GenericTokenNodeType("INTEGER_LITERAL", 1176, "000")` or `new GenericTokenNodeType("CHARACTER_LITERAL", 1178, "'c'")`, where the numbers are the unique index of the node type, and the string is the token representation, used in `GetSampleText` and `GetDescription`.
