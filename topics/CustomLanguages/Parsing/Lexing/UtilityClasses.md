[//]: # (title: Utility Classes)

ReSharper includes a couple of classes that are useful when writing a lexer, namely `NodeTypeSet` and `LexerDictionary<T>`. The `NodeTypeSet` class the lexer can use to test if a `NodeType` instance is part of a particular set of nodes, such as keywords, symbols and so on. The `LexerDictionary` class is a dictionary of strings to `NodeType`, optimised for use in a lexer.

## NodeTypeSet

The `NodeTypeSet` class is very simple; it is a set of `NodeType` instances. A custom language can create one or more static instances of `NodeTypeSet` to represent specific sets of node types, such as keywords, symbolic operators, literals or any other set that makes sense for the language. A lexer, or indeed parser or other piece of code, can see if a `NodeType` is in the set, as a quick and efficient lookup to categorise the node type, or tree element.

The `NodeTypeSet` itself derives from `NodeTypeDictionary<bool>`, which is a simple dictionary like class that returns `true` if the `NodeType` is in the set and `false` if it isn't. The check is performed using the indexer, such as:

```csharp
if (myNodeTypeSet[CSharpTokenType.ABSTRACT_KEYWORD])
{
  // ...
}
```

Creating a `NodeTypeSet` is easy - simply pass all of the tokens into the constructor:

```csharp
public static readonly NodeTypeSet KEYWORDS = new NodeTypeSet(
  CSharpTokenType.ABSTRACT_KEYWORD,
  CSharpTokenType.AS_KEYWORD,
  CSharpTokenType.BASE_KEYWORD,
  ...
);
```

The class also includes a number of typical set methods:

```csharp
public class NodeTypeSet : NodeTypeDictionary<bool>, IEnumerable<NodeType>
{
  // ...
  public NodeTypeSet Except(NodeTypeSet set);
  public NodeTypeSet Except(NodeType nodeType);
  public NodeTypeSet Union(NodeTypeSet set);
  public NodeTypeSet Union(NodeType nodeType);
  public NodeTypeSet Intersect(NodeTypeSet set);
}
```

## LexerDictionary

The `LexerDictionary<T>` class is a memory-efficient means of looking up an object from a string in a lexer's buffer, without allocating any memory for the string. It will typically be used to map a string to a `NodeType` or `TokenNodeType` instance.

The main use case for `LexerDictionary` is while lexing keywords and identifiers. In many languages, keywords are simply reserved identifiers. E.g. the rules that match the identifier `"hello`" would also match `"return"`, if it wasn't reserved as a keyword.

When implementing a lexer, keywords can either be explicitly added as standalone rules, or matched as identifiers, and checked against reserved keywords. The `LexerDictionary` is designed to make that easy - if the token text is in the "keywords" dictionary, it's a keyword, otherwise, it's just an identifier.

```csharp
public partial class MyLexer
{
  public static LexerDictionary<TokenNodeType> myKeywords = new LexerDictionary<TokenNodeType>();

  public static MyLexer()
  {
    foreach (var tokenNodeType in KEYWORDS.OfType(typeof(TokenNodeType))
      myKeywords[tokenNodeType.TokenRepresentation] = tokenNodeType;
  }

  private TokenNodeType FindKeywordByCurrentToken()
  {
    return keywords.GetValueSafe(myBuffer, yy_buffer, yy_buffer_start, yy_buffer_end);
  }
}
```

And then in the [CsLex](CsLex.md) rules:

```
<YYINITIAL> {IDENTIFIER}   { return makeToken(FindKeywordByCurrentToken() ?? CSharpTokenType.IDENTIFIER); }
```

The `GetValueSafe` method will use the string at the current location in the lexer's buffer to look up the underlying object (`TokenNodeType` in this example), returning `null` if not found. It does this without allocating any strings for the given buffer range's text, which is essential when the lexer is called as frequently as a user types.

The class derives from `Dictionary<object, T>`, which means that the key is actually an `object`. Items are usually added with a `string` key, but are then looked up using a buffer, an offset and a length. No strings are available or allocated during lookup.

The dictionary is created with an instance of `TokenTextEqualityComparer`, which is an implementation of `IEqualityComparer<object>` that the dictionary will use to check for equality of keys. This equality comparer can handle multiple types of key, and will perform a string comparison using the characters of a buffer without having to allocate a string.

The equality comparer handles instances of `ILexer`, `BufferRange` and `ReusableBufferRange`, as well as `string`. If given an instance of `ILexer`, it will use the `Buffer`, `TokenStart` and `TokenEnd` properties to get at the characters of the buffer.

The `BufferRange` object contains a reference to `IBuffer` and a `TextRange` for the start and end character offsets. It is a value type, which means it is cheap to allocate and doesn't cause garbage collection, however, if passed to the equality comparer, it causes allocations through boxing. A common pattern is for a lexer to allocate a single instance of `ReusableBufferRange` in its constructor. This contains a `BufferRange` that can be updated in place, without any allocations. The lexer itself is single threaded, so a single instance is safe for use in lookups.

The `LexerDictionary.GetValueSafe` method takes a `ReusableBufferRange`, and updates it (in place) before passing it to `LexerDictionary`. Again, there are no allocations:

```csharp
public partial class MyLexer
{
  private readonly ReusableBufferRange myBuffer = new ReusableBufferRange();

  private TokenNodeType FindKeywordByCurrentToken()
  {
    // LexerDictionary.GetValueSafe will call myBuffer.Reuse(buffer, new TextRange(start, end))
    // which will update the ReusableBufferRange without allocating any memory
    // (the TextRange is a struct, so copied in place)
    return keywords.GetValueSafe(myBuffer, yy_buffer, yy_buffer_start, yy_buffer_end);
  }
}
```

By default, the `LexerDictionary` is case sensitive, but this can be made case insensitive by passing `false` to the constructor.
