---
title: Implementing a lexer
---

ReSharper requires a custom language to create a lexer that implements (at least) the `ILexer` interface:

```csharp
public interface ILexer
{
  void Start();
  void Advance();
  object CurrentPosition { get; set; }

  TokenNodeType TokenType { get; }
  int TokenStart { get; }
  int TokenEnd { get; }

  IBuffer Buffer { get; }
}
```

The [`IBuffer`](TextBuffers.md) is given to the lexer in the constructor, via `ILexerFactory`.

Clients of the lexer will follow these steps:

* Call `Start` to get the lexer to recognise the first token.
* Retrieve the current token type from the `TokenType` property, which will be a singleton instance of a language-specific class that derives from `TokenNodeType` (see the guide on [Token Node Types](../NodeTypes/TokenNodeTypes.md) for more details).
* Use the `TokenStart` and `TokenEnd` properties to retrieve the offset of the token start and end in the text buffer. This is required because the token type is a singleton instance, and therefore cannot contain details about the location and length of the token itself. The start offset is inclusive, and the end offset is exclusive, just like [`TextRange`](TextBuffers.md#text-range) (e.g. given "Hello world", the text range (0, 5) returns "Hello").
* Call the `Advance` method repeatedly, to move to the next token, which will update the `TokenType`, `TokenStart` and `TokenEnd` properties with the information about the current token and location.

The `CurrentPosition` property is a lexer specific object that encapsulates the information required by the lexer to save and restore the current location. The `LexerStateCookie` class can be used by parsers to make it easy to rollback to a specific state in the lexer. It implements the `IDisposable` interface, so it can be used in `using` statement:

```csharp
using(LexerStateCookie.Create(myLexer))
{
  myLexer.Advance();
  // ...
} // Lexer position is restored
```

This can be used to implement lookahead, retrieving a number of tokens ahead, then rolling back to the current position (see [Lexer Utility Methods](LexerUtil.md) for more details).

## Strongly typed lexers

The `ILexer` class exposes the `CurrentPosition` as an object, to allow lexers maximum flexibility for storing state about the current position - the lexer can return any object it wishes. However, if the lexer wishes to return a value type, this can add boxing allocations, so the lexer can also implement `ILexer<TState>`:

```csharp
public interface ILexer<TState> : ILexer
{
  new TState CurrentPosition { get; set; }
}
```

This overrides (shadows) the `CurrentPosition` property to be of type `TState` instead of `object`. This will allow a value type to be returned without boxing allocations. For example, if the lexer only requires an integer position (such as a [caching lexer](CachingLexers.md)), it can implement `ILexer<int>`, and avoid boxing the `int` into `object`.

Similarly, the lexer can implement its state object as a `struct`, and return it as a strongly typed item:

```csharp
public struct CSharpLexerState
{
  public TokenNodeType currTokenType;
  public ImmutableStack<CSharpLexerStackItem> stack;
  public int yy_buffer_index;
  public int yy_buffer_start;
  public int yy_buffer_end;
  public int yy_lexical_state;
}

public ILexer<CSharpLexerState>.CurrentPosition
{
  get
  {
    CSharpLexerState csharpLexerState;
    csharpLexerState.currTokenType = ...;
    // ...[snip]...
    return csharpLexerState;
  }
  set { /* ... */ }
}
```

The `struct` is copied by value to the caller of `CurrentPosition`, and no boxing allocations take place.

## Incremental lexers

ReSharper includes infrastructure for incremental lexing, that is, only lexing the parts of a file that change, and reusing existing tokens for the rest of the file. Most of the work is handled by [a caching lexer](CachingLexers.md), and is covered in more detail in the section on incremental parsing.

The custom language parser can implement the `ILexerEx` and `IIncrementalLexer` interfaces:

```csharp
public interface ILexerEx : ILexer
{
  uint LexerStateEx { get; }
}

public interface IIncrementalLexer : ILexerEx, ILexer
{
  int EOFPos { get; }

  // Number of lexems that incremental re-lexing should step back to start relexing
  int LexemIndent { get; }

  void Start(int startOffset, int endOffset, uint state);
}
```

These interfaces expose the lexer state as a `uint` value. If the lexer is built with [CsLex](CsLex.md), this state can be the `yy_lexical_state` value, which is used to decide when specific regular expression rules are applied. Alternatively, it can be used as a lookup into other (static) values, or used to encode more state information into the bits of the `uint` (the C# lexer uses this strategy to encode a stack of items).

The `IIncrementalLexer` interface has a `Start` method, which allows the lexer to start from an arbitrary point in the text buffer, without having to parse the preceding part of the file first. It takes a start and end offset, and also the `uint` state value returned from `ILexerEx.LexerStateEx`. These values will have been cached from a previous scan of the text buffer. The `TokenBuffer` and `CachingLexer` classes implement this.

More details are in the section on incremental parsing.
