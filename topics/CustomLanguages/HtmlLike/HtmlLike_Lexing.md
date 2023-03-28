[//]: # (title: Lexing HTML-Like Languages)

Lexing an HTML-like language can be complicated - it must handle existing HTML syntax, but also provide support for new syntax, which might be new tags, new tag syntax (`<%` and `%>`), start block delimiters (Razor's `@`, which has no closing delimiter) and even nested HTML inside a language construct (Razor again). And it shouldn't have to reimplement HTML.

## Compound lexer

ReSharper manages this using `HtmlCompoundLexer<TLexer>` and `HtmlCompoundIncrementalLexer<TLexer>`. These are implementations of `ILexerEx` and `IIncrementalLexer` that switches between a lexer for the custom syntax and a lexer for standard HTML. The `TLexer` generic parameter is the lexer for the custom syntax, and must implement the `IHtmlCompoundLexer` interface.

### Creating the compound lexer

The constructor looks like this:

```csharp
public HtmlCompoundLexer(IBuffer buffer, TLexer baseLexer, IIncrementalLexerFactory htmlLexerFactory)
```

* `buffer` represents the text of the file to be lexed.
* `baseLexer` is an instance of the lexer used to parse the custom HTML syntax, and is of the generic type `TLexer`.
* `htmlLexerFactory` is a factory class that is used to create the standard HTML lexer.

Typically, this is called from the `CreateLexer` method of a HTML-like language's `LanguageService`:

```csharp
public ILexer CreateLexer(IBuffer buffer)
{
  return new HtmlCompoundLexer(buffer, new MyHtmlLikeSyntaxLexer(myTokenNodeTypes, buffer),
    new HtmlLexerFactory(myTokenNodeTypes));
}
```

A new instance of the lexer for the custom syntax is passed in, as is a new instance of the standard `HtmlLexerFactory`. Both classes take a parameter of `myTokenNodeTypes`, which a [per-language component](Registration_PerLanguageComponents.md) that derives from `IHtmlComponentTokenNodeTypes`:

```csharp
public interface IHtmlCompoundTokenNodeTypes : IHtmlTokenNodeTypes
{
  [NotNull] TokenNodeType RAW_HTML { get; }
}
```

The `RAW_HTML` property is used by the compound lexer to know when to switch between the custom lexer and the standard HTML lexer. See the [HTML Token Node Types]() section for more details on this interface and implementing token node types.

### Running the compound lexer

The compound lexer is used just like a normal implementation of `ILexer` - `Advance()` is called multiple times until the end of file.

The base (custom) lexer drives the lexing of the file - the compound lexer will initially use the tokens of the base lexer. However, if the base lexer encounters a `RAW_HTML` token, the compound lexer will switch to the HTML lexer, and use the HTML tokens.

This means:

* Initially, advance the base lexer.
* Update the compound lexer's own state (`ILexerEx.LexerStateEx`) by combining the states of the base leer, the HTML lexer and a flag to indicate if the HTML lexer is currently running. This state can be saved and restore for incremental lexing.
* Compare the base lexer's current token to the custom language's `IHtmlCompoundTokenNodeTypes.RAW_HTML`:
    * If different, use the base lexer's current token.
    * If the same, reset the (incremental) standard HTML lexer over the span of the `RAW_HTML` token and set the flag to indicate the HTML lexer is running.
* On subsequent calls to `Advance`, if the flag is set, advance the HTML lexer and use that token, otherwise, advance the base lexer.
* Once the span of the `RAW_HTML` token has been lexed (by the HTML lexer), reset the HTML running flag and start advancing the base lexer again.

### Incremental lexing

The `HtmlCompoundIncrementalLexer` class is essentially the same as `HtmlCompoundLexer` except that it also implements `IIncrementalLexer`, allowing for incremental lexing. More details on incremental lexing can be found in the topics on [Implementing a Lexer](ImplementingLexers.md), [Caching Lexers](CachingLexers.md) and incremental parsing.

## Custom syntax lexer

The base lexer is intended to create tokens for the custom language syntax. It must implement `IHtmlCompoundLexer`:

```csharp
public interface IHtmlCompoundLexer : ILexerEx
{
  IHtmlCompoundTokenNodeTypes TokenTypes { get; }
  short NStates { get; }
}
```

The `TokenTypes` property returns the per-language component that implements `IHtmlCompoundTokenNodeTypes`, so that the compound lexer can access the `RAW_HTML` property, which is the token node type the custom lexer uses to indicate a block of raw HTML. This is a different token node type for each HTML-like custom language.

The `NStates` property is the number of states supported by the custom lexer. In other words, this is the maximum value that will be set in `ILexerEx.LexerStateEx` for the custom lexer. It is used when combining the base lexer and HTML lexer states for incremental lexing in the compound lexer.

If built with [CsLex](CsLex.md), this will be the number of states such as `YYINITIAL` used in the lexer definition, and can be implemented like:

```csharp
public short NStates { get { return (short) yy_state_dtrans.Length; } }
```

### Tips for implementing the base lexer

* The base lexer should create tokens for the custom language as normal, and should return `RAW_HTML` for any chunks of HTML.
* If the custom language's syntax is valid HTML, let the standard HTML lexer/parser handle it. For example, WebForms' `<asp:Content runat="server" ...` is valid HTML, and is lexed and parsed as HTML.
* An embedded HTML block should start with valid HTML syntax. The HTML lexer starts with empty state, so the first `RAW_HTML` block must start with valid HTML.
* The HTML lexer state is preserved across `RAW_HTML` blocks, so it is possible to create "nested" blocks of custom syntax inside HTML. For example:

    ```html
    <div class="@MyClass" />
    ```

    This could be implemented as a `RAW_HTML` token that spans from the start of the element up to the `@` symbol - `<div class="`. The HTML lexer will then return tokens for the opening bracket, the element, the attribute name, the equals, the quote symbol and then relinquish control to the base lexer, which will return tokens for the expression - `@MyClass`. It can then return `RAW_HTML` for the closing quotes and the closing element bracket - `" />`. Because the HTML lexer state is preserved, HTML lexing will continue as expected, and the HTML tokens are correct.
