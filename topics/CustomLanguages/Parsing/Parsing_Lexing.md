[//]: # (title: Lexing)

[Lexical analysis](https://en.wikipedia.org/wiki/Lexical_analysis) (or "lexing") is the process of converting a text buffer into a sequence of tokens which uniquely identify the building blocks of a custom language. For example, a lexer can convert the C# expression `age = 42;` into the following stream of tokens:

```text
IDENTIFIER
WHITESPACE
EQUALS
WHITESPACE
INTEGER_LITERAL
SEMI_COLON
```

A parser can then analyse this stream of tokens, looking for known patterns, and build a parse tree that identifies an assignment operation, with an identifier and an constant expression.

This section looks at how ReSharper expects to be able to create a lexer via a [lexer factory](LexerFactories.md), and the lexer itself must [implement the `ILexer` interface](ImplementingLexers.md), and use [text buffers](TextBuffers.md) to access the text block to analyse. This section also takes a look at how to [use the CsLex SDK tool to tokenise a custom language](CsLex.md).

Finally, this section takes a look at a couple of useful features ReSharper uses while parsing - [lexer utility methods](LexerUtil.md), [filtering lexers](FilteringLexers.md), and [caching lexers](CachingLexers.md).
