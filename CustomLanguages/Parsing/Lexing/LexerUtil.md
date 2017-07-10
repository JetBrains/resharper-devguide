---
title: Lexer Utility Methods
---

The `LexerUtil` class provides a number of utility and extension functions for lexing and parsing. Of particular interest are the `LookaheadToken` methods, `GetTokenLength` and `GetCurrTokenText`:

```csharp
public static class LexerUtil
{
  // Look ahead to the k'th token and return it, the roll back to the current position
  static public TokenNodeType LookaheadToken(this ILexer lexer, int k);
  static public TokenNodeType LookaheadToken<T>(this ILexer<T> lexer, int k);
  // Look ahead, but skip the given node type
  static public TokenNodeType LookaheadTokenSkipping(this ILexer lexer, int k, TokenNodeType nodeTypeToSkip);

  // Get the length of the text of the current token
  public static int GetTokenLength(this ILexer lexer);

  // Get the text of the current token, with or without quotes
  public static string GetCurrTokenText(this ILexer lexer);
  public static string GetQuotedTokenText(this ILexer lexer, char quote);
  public static string GetQuotedTokenText(this ILexer lexer, char openQuote, char closeQuote);

  // Compare the current token text against the given string. Functionally equivalent
  public static bool CompareTokenText(ILexer lexer, string str, bool caseSensitive = true);
  public static bool CompareBufferText(ILexer lexer, string str, bool caseSensitive = true);

  // Return a lazy enumerable of TokenNodeType by calling lexer.Advance
  public static IEnumerable<TokenNodeType> Tokens(this ILexer lexer);

  // Advance lexer while the current token is the same as the skip token(s)
  public static int AdvanceWhile<TLexer>(this TLexer lexer, TokenNodeType skipToken);
  public static int AdvanceWhile<TLexer>(this TLexer lexer, NodeTypeSet skipTokens);
}
```
