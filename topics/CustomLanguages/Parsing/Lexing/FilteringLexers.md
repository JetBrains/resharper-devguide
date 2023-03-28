[//]: # (title: Filtering Lexers)

Parsers are designed to recognise specific sequences of tokens, and if that sequence contains lots of whitespace or comments, or other syntactically and semantically insignificant tokens, then it makes recognising patterns much harder.

A filtering lexer will filter out whitespace, comments and other insignificant tokens, allowing the parser a "clean" sequence of tokens that make it easier to pattern match against.

Each language must implement a filtering lexer, and expose it by implementing the `LanguageService.CreateFilteringLexer` abstract method. This method is used in several places in the codebase, but typically, a language will use a filtering lexer when creating its own parser.

Implementing a filtering lexer is very straightforward, simply derive from the `FilteringLexer` or `FilteringLexerBase` abstract base classes. The `FilteringLexerBase` class implements `ILexer` by delegating to an existing `ILexer` passed into the constructor:

```csharp
public abstract class FilteringLexerBase : ILexer
{
  protected FilteringLexerBase(ILexer lexer)
  {
    // ...
  }

  // ILexer.Advance
  public virtual void Advance()
  {
    myInterruptChecker.CheckForInterrupt();
    myLexer.Advance();
    SkipFilteredTokens();
  }

  // ILexer.Start
  public virtual void Start()
  {
    myLexer.Start();
    SkipFilteredTokens();
  }

  // ILexer implementation snipped...

  protected abstract void SkipFilteredTokens();
  protected abstract book Skip(TokenNodeType tokenType);
}
```

It also requires an abstract method `SkipFilteredTokens` to be implemented. This method is called to move the underlying lexer forward, passed any tokens that should be filtered out and skipped. The `Skip` abstract method is intended to be a companion to the implementation of `SkipFilteredTokens`, and will return `true` if a given `TokenNodeType` is to be skipped.

The `FilteringLexer` abstract base class provides a simple implementation of `SkipFilteredNodes`, which is sufficient for most requirements, but requires `Skip` to be implemented by the deriving class:

```csharp
public abstract class FilteringLexer : FilteringLexerBase
{
  protected override SkipFilteredTokens()
  {
    TokenNodeType tokenType = myLexer.TokenType;
    while (tokenType != null && Skip(tokenType))
    {
      myInterruptChecker.CheckForInterrupt();
      myLexer.Advance();
      tokenType = myLexer.TokenType;
    }
  }
}
```

Finally, the `Skip` method can be implemented by the deriving class. For example, the CSS filtering lexer, simply checks to see if the token is a comment:

```csharp
public class CssFilteringLexerBase : FilteringLexer
{
  public CssFilteringLexerBase(ILexer lexer) : base(lexer)
  {
  }

  protected override bool Skip(TokenNodeType tokenType)
  {
    return tokenType.IsComment;
  }
}
```

## Checking for interrupts

One thing to notice in the filtering lexer is the use of `myInterruptChecker.CheckForInterrupt()`. Lexing needs to be quick, and needs to be updated whenever the user types in the editor. It is inefficient to continue lexing when the text buffer has changed, so the `FilteringLexerBase` creates a new instance of `SeldomInterruptChecker` and calls it periodically to see if the process has been interrupted, and should be aborted. If so, the `SeldomInterruptChecker` will throw an instance of `ProcessCancelledException`. There is no need to catch this exception, it will be handled by ReSharper. The `SeldomInterruptChecker` uses the `InterruptableActivityCookie` class to monitor standard interruptions.

Typically, a language's lexer and parser don't check for interrupts themselves. This is usually handled by the filtering lexer.

## Inserting missing tokens

When the parser builds a PSI tree, the resulting tree needs to cover the entire contents of the text buffer. Using a filtering lexer means that any tokens that are filtered out are not added to the PSI tree, and it contains holes.

After parsing with a filtering lexer, a *missing token inserter* must be run to reinsert these missing tokens. The `MissingTokenInserterBase` abstract base class provides a base implementation, by walking the PSI tree and calling the `ProcessLeafElement` abstract method for each leaf element.

More details can be found in the Parsers section.
