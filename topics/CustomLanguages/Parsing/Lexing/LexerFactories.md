[//]: # (title: Lexer Factories)

ReSharper requires a custom language to implement `ILexerFactory`, which is exposed via `IProjectFileLanguageService.GetMixedLexerFactory` and the `LanguageService.GetPrimaryLexerFactory` abstract method.

The lexer factory is assigned to the `IFile` root node of the PSI tree, via the `IFileImpl` interface, which exposes implementation details rather than PSI tree information. This is made available so that various parts of ReSharper can easily create a fresh lexer on a new buffer.

```csharp
public interface ILexerFactory
{
  ILexer CreateLexer(IBuffer buffer);
}
```

This class is usually trivial to implement, simply returning a new instance of a class that implements the `ILexer` interface.

 >  The `LanguageService.GetPrimaryLexerFactory` method should be implemented by creating a new instance of this class, and typically, the `IProjectFileLanguageService.GetMixedLexerFactory` will just call `GetPrimaryLexerFactory`.
>
> However, if the custom language hosts "secondary" PSI trees (such as HTML, which can also contain CSS and JavaScript), the `IProjectFileLanguageService` should inherit form `MixedProjectLanguageService`, which provides an implementation of `GetMixedLexerFactory` that returns an instance of `MixedLexerFactory` using the lexer factory returned from `GetPrimaryLexerFactory`.
 >
 {type="note"}

The `IBuffer` interface is an abstraction of a text based buffer that a lexer uses to get the contents of the file (persistent or in-memory). See the guide on [Text Buffers](TextBuffers.md) for more details.