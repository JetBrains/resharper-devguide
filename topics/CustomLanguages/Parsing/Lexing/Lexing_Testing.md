[//]: # (title: Testing Lexers)

The `LexerTestBase` class can be used to create tests for lexers. It derives from `BaseTestWithSingleProject`, so it creates an in-memory ReSharper instance with a solution loaded that contains a single project. The project will consist of the file or files named in the test methods.

The input file, that gets added to the in-memory project, should be an example file in the custom language. The `LexerTestBase` class will create an instance of the language's `ILexer`, with the default implementation using the file extension to get an instance of the custom language's `IProjectFileLanguageService`, and then calling `GetMixedLexerFactory`.

The entire example file is then lexed, and the output file is compared against the gold file to ensure it matches. The output file is a stream of token types, one on each line. The value of the token type is the `ToString` implementation of the `TokenNodeType` instance, which is either the token's representation, or it's identifier (see [token node types](TokenNodeTypes.md) for more details on the representation and identifier).

For example, given the following CSS file - `test01.css`:

```css
h1 { color: red }
```

Then the `test01.css.gold` file would be:

```text
IDENTIFIER
WHITE_SPACE
LBRACE
WHITE_SPACE
IDENTIFIER
COLON
WHITE_SPACE
IDENTIFIER
WHITE_SPACE
RBRACE
```

## Implementation

Derived classes will need to override `RelativeTestDataPath` to declare the path to the data and gold files, relative to the `test\data` folder. 

Each test method can call one of the following methods, which allows specifying the base name used to create the input and gold file names:

* `DoNamedTest` or `DoNamedTest2` - will use the name of the test method as the file base name. The `DoNamedTest2` method will also strip any `Test` prefix from the method first.
* `DoOneTest` or `DoTestFile` will use the passed in string as the file base name. `DoOneTest` simply calls `DoTestFile`.

The input file name that gets added to the in-memory project is created from the test's base file name, plus the value of the `Extension` property. This retrieves the extension from any attributes on the test method or class that implement `ITestFileExtensionProvider`. Typically, this is achieved by using the `[TestFileExtension]` attribute, and specifying the extension (with dot) in the constructor. This is usually taken as a constant value defined in the [project file type](ProjectFileType.md):

```csharp
[TestFileExtension(CssProjectFileType.CSS_EXTENSION)]
public class CssLexerTest : LexerTestBase
{
  protected override String RelativeTestDataPath  { get { return @"lexing\css"; } }

  [Test] public void nthTest01() { DoNamedTest(); }
  // ...[snip]...
}
```

## Customisation

The `LexerTestBase` class allows for the behaviour of the class to be customised, by overriding various methods:

* `CreateLexer` - the default implementation of `CreateLexer` will create an instance based on the file extension of the test file, and using the registered project file type's `IProjectFileLanguageService.GetMixedLexerFactory`.

    The default implementation - `CreateLexer(IProjectFile pf, StreamReader sr)` can be overridden directly, or the `CreateLexer(StreamReader sr)` helper method can be overridden instead, to simply create a new lexer based on the contents of the passed in stream reader:

```csharp
protected virtual ILexer CreateLexer(StreamReader sr)
{
  return new CSharpLexer(new StringBuffer(sr.ReadToEnd()));
}
```

* `WriteToken` is used to output the current `TokenNodeType` while lexing the file. It will call the default `ToString` implementation, but can be overridden to provide more detail, if required. For example, this override will also output the token start and end offsets:

```csharp
protected override void WriteToken(TextWriter writer, ILexer lexer)
{
  writer.WriteLine("{0} [{1}-{2}]", lexer.TokenType, lexer.TokenStart, lexer.TokenEnd);
}
```
