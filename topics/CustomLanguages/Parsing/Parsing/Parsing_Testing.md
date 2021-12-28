[//]: # (title: Testing Parsers)

The `ParserTestBase<TLanguage>` can be used to create tests for a parser of a specific language. It derives from `BaseTestWithTextControl`, which means it will create an in-memory ReSharper instance, with a solution and a project loaded. The test will load a specific text file into an in-memory text control, parse it, retrieve any `IFile` instances that match `TLanguage`, and compare the results against a `.gold` file.

The `.gold` file will be the `IFile` and children, written as an indented text file. Each line will be another node in the tree, with the text being the result of calling `ToString()` on the node instance.

For example, given the following CSS file - `test01.css`:

```css
/* {caret} */
body
{
  margin: 0;
}
```

Then the `test01.css.gold` file would be something like:

```
ICssFile
  CssComment(type:COMMENT, text:/* {caret} */)
  CssNewLineToken(type:NEW_LINE, text:\n) spaces:"\n"
  ICssFileSection
    IRuleset
      IRulesetDeclarationList
        IRulesetDeclaration:body
          ISimpleSelector:body
            ITypeSelector:body
              CssIdentifierToken(type:IDENTIFIER, text:body)
      CssNewLineToken(type:NEW_LINE, text:\n) spaces:"\n"
      ICssPropertyBlock
        CssGenericToken(type:LBRACE, text:{)
        CssNewLineToken(type:NEW_LINE, text:\n) spaces:"\n"
        CssWhitespaceToken(type:WHITE_SPACE, text:  ) spaces:"  "
        ICssPropertyStatement
          ICssProperty
            CssIdentifierToken(type:IDENTIFIER, text:margin)
            CssGenericToken(type:COLON, text::)
            CssWhitespaceToken(type:WHITE_SPACE, text: ) spaces:" "
            ICssPropertyValue
              ICssLiteralExpression
                CssLiteralToken(type:INTEGER_LITERAL, text:0)
          CssGenericToken(type:SEMICOLON, text:;)
        CssNewLineToken(type:NEW_LINE, text:\n) spaces:"\n"
        CssGenericToken(type:RBRACE, text:})
```

Note that the PSI tree includes comments and whitespace.

Also note that the input file contains a `{caret}` text. The `ParserTestBase<T>` class finds the file to test by looking for the current text caret location, as defined by special strings in the file. Here, it is looking for `{caret}` to know which file should be processed. This means that all input files require the `{caret}` text to be added to the file. See below for an alternative implementation that doesn't require `{caret}` in the input file.

## Implementation

Derived classes will need to override `RelativeTestDataPath` to declare the path to the data and gold files, relative to `test\data`.

Each test method can call one of the following methods, which allows specifying the base name used to create the input and gold file names:

* `DoNamedTest` or `DoNamedTest2` - will use the name of the test method as the file base name. The `DoNamedTest2` method will also strip any `Test` prefix from the method first.
* `DoOneTest` will use the passed in string as the file base name.

These methods will specify the base name of the file to add to the in-memory project, but do not specify the file extension. This is added by the test base classes, by appending the value of the `Extension` property. This property will retrieve the extension to use by looking for attributes on either the test method or the test class. The first attribute that implements `ITestFileExtensionProvider` is used to get the value. Typically, this is achieved by using the `[TestFileExtension]` attribute, and specifying the extension (with dot) in the constructor. This is usually taken as a constant value defined in the [project file type](ProjectFileType.md).

```csharp
[TestFileExtension(CssProjectFileType.CSS_EXTENSION)]
public class CssParserTest : ParserTestBase<CssLanguage>
{
  protected override string RelativeTestDataPath { get { return @"parsing\css"; } }

  [Test] public void test01() { DoNamedTest(); }
  // ...[snip]...
}
```

## Customisation

The `ParserTestBase<T>` base class does not provide any hooks to customise. However, it is possible to use an alternative implementation, which does not require the `{caret}` special text in the input file, and includes extra asserts.

## Alternative implementation

The default base class requires the input file to include `{caret}` so that it knows which file to test, even if there is only one file added to the in-memory project. As an alternative, a language can implement a different base class that doesn't have this requirement, something like this:

```csharp
public abstract class AlternativeParserBaseTest<TLanguage> : BaseTestWithSingleProject
{
  protected abstract TLanguage GetLanguage();

  protected override void DoTest(IProject testProject)
  {
    var projectItem = testProject.GetSubItems().Single();
    string buffer = projectItem.Location.ReadAllText(Encoding.Default);

    var languageService = GetLanguage().LanguageService();
    Assert.IsNotNull(languageService);

    var lexer = languageService.CreateCachingLexer(new StringBuffer(buffer.Trim()));
    var psiModule = Solution.PsiModules().GetPrimaryPsiModule(testProject, TargetFrameworkId.Default);
    Assert.IsNotNull(psiModule);
    var parser = languageService.CreateParser(lexer, psiModule, null);
    var psiFile = parser.ParseFile();
    Assert.IsNotNull(psiFile);
    SandBox.CreateSandBoxFor(psiFile, psiModule);

    ExecuteWithGold(projectItem.Name, writer => DebugUtil.DumpPsi(writer, psiFile));
    Assert.AreEqual(buffer.Trim(), psiFile.GetText(), "Reconstructed text mismatch");
    CheckRange();
  }

  private static void CheckRange (string documentText, ITreeNode node)
  {
    Assert.AreEqual(node.GetText(), documentText.Substring(node.GetTreeStartOffset().Offset, node.GetTextLength()), "node range text mismatch");

    for (var child = node.FirstChild; child != null; child = child.NextSibling)
      CheckRange(documentText, child);
  }
}
```

This will create a parser for the single file in the project, retrieve the `IFile`, and compare it against the `.gold` file. The input file does not require the `{caret}` identifier.

Furthermore, this test will also check that the text of the file matches the text recreated by getting the text of each node in the tree, as well as validating the range of each tree node against the length of its text, further ensuring the tree is correct.