[//]: # (title: Extended HTML Parsing)

Custom HTML-like languages can modify the lexing of standard HTML by providing an alternative HTML lexer to the constructor of `HtmlCompoundLexer` (typically a new instance of `HtmlLexerFactory` is passed).

However, it is not currently possible to extend the lexing/parsing of standard HTML files, or the standard HTML portions of custom files such as WebForms or Razor, from third party code. An example of this would be to add support for custom templates inside the language, such as Angular's interpolation expressions:

```html
<button disabled="{{isDisabled}}">{{disabledValue}}</button>
```

This kind of syntax can be supported by third party code using existing extension mechanisms. For example, if the syntax is simple enough, this could be implemented by [References](References.md) and [Daemon](Analysis.md) support for highlighters. If the expression is more complex, it can also be implemented by using [Injected PSI](InjectedPsi.md), although this is not very efficient, as the content of the nodes are converted to and from strings on modification. There are also limitations on nested injections. It is best to only use injected PSI for small ranges.

## Angular expressions

ReSharper's support for Angular expressions is currently implemented differently, to work around these limitations. The implementation of `HtmlProjectFileLanguageService` returns `HtmlLanguage` by default, but returns `Angular2HtmlLanguage` when Angular support is enabled.

This means that HTML files are treated as "Angular HTML" files, and are now associated with the `Angular2HtmlLanguage` PSI language type. It also means that the per-language components for HTML files are resolved using `Angular2HtmlLanguage` instead of `HtmlLanguage`. One implication of this is that the lexer factory registered for HTML files becomes `Angular2HtmlLexerFactory` instead of `HtmlLexerFactory`. This new lexer factory creates a lexer that is derived from `HtmlRawLexerGenerated`, and parses standard HTML, but also handles expressions that are wrapped in braces.

This does not affect compound lexers - typically, they pass in a new instance of `HtmlLexerFactory`. To get Angular support in a custom HTML-like language, pass in an instance of `Angular2HtmlLexerFactory`.

In order to match ReSharper behaviour, the custom language should check to see if Angular support is enabled, by taking in an instance of the `AngularIsEnabledProvider` shell component, and comparing the `GlobalAngularLevel` to `AngularLevel.None`.