[//]: # (title: Testing Daemon Stages and Highlights)

The `HighlightingTestBase` base class provides infrastructure for testing daemons, element problem analysers and highlightings. It creates an in-memory project, adds a file and analyses it. The resulting highlights are then written to a `.tmp` file and compared against the `.gold` file. If the files differ, the test fails.

Further derived classes, such as `CssHighlightingTestBase`, `CSharpHighlightingTestBase`, `CSharpHighlightingTestNet45Base`, etc. provide extra support for language specific tests.



## Input and gold files

The input file is a simple text file. The location of the text caret is irrelevant to highlight tests, so placeholders are not required. As an example testing C# local variable usage:

```csharp
class C
{
  public void foo (bool cond)
  {
    int counter = 0;
    while(cond)
      counter++;
  }
}
```

By default, the gold file will look something like this:

```csharp
class C
{
  public void foo (bool cond)
  {
    |int|(0) |counter|(1) = 0;
    while(|cond|(2))
      counter++;
  }
}
------------------------------------------------------(0): ReSharper Hint: Use implicitly typed local variable declaration
(1): ReSharper Warning [CS0219]: Local variable 'counter' is only assigned but its value is never used
(2): ReSharper Warning: Loop control variable is never changed inside loop
```

The input file is replicated at the top of the gold file, with the location of highlights marked up using the `|text|(index)` format. The text that the highlight is applied to is delimited with pipe characters `|`, with the index of the highlight listed after, in brackets.

The text of the highlight, which would normally be displayed as a tooltip or in the status bar, is added to the end of the file, after a separator. Each highlight's index is listed first.

It is possible to change the output of the highlight text by overriding the `CreateHighlightingDumper` method and returning a custom instance of `TestHighlightingDumper`.

## Using the base class

In order to use the base class, simply derive from `HighlightingTestBase`, or one of the more language specific derived class (see below for more details on these derived classes), and call `DoTestSolution`, `DoNamedTest` or `DoNamedTest2` in the test method. As ever, the `RelativeTestDataPath` should be overridden to describe where the test data files are located. Most of the language specific base classes set up a value of `Daemon\<Language>`.

```csharp
[Test]
public void DoTest()
{
  // Runs a specific set of files
  DoTestSolution("Foo.cs");
}

[Test]
public void CheckHighlights()
{
  // File name is based on the name of the test,
  // e.g. "CheckHighlights.cs"
  DoNamedTest();
}

[Test]
public void TestLocalVariableUsage()
{
  // File name is based on the name of the test, stripping a "Test" prefix
  // e.g. "LocalVariableUsage.cs"
  DoNamedTest2();
}
```

The `DoNamedTest` and `DoNamedTest2` methods automatically add an extension in order to find the input file. This extension comes from the `Extension` property, which looks for attributes on the test method or class that implement `ITestFileExtensionProvider`, such as `TestFileExtensionAttribute`. It defaults to `.cs`.

Additional files can be passed to either `DoTestSolution` and `DoNamedTest`/`DoNamedTest2`. This allows for testing a single file while also allowing other files to produce a compilable project.

### Testing with combinations of settings

The `HighlightinTestBase` class understands iterating over multiple combinations of settings. In order to make use of this, please refer to the section on [settings iteration](OptionsIterator.md).

### Filtering highlights

The test class can filter what highlights it wished to test, by overriding the `HighlightingPredicate` method.

```csharp
protected virtual bool HighlightingPredicate(IHighlighting highlighting, IContextBoundSettingsStore settingsStore)
{
  var manager = HighlightingSettingsManager.Instance;
  var severity = manager.GetSeverity(highlighting, settingsStore);
  var attribute = manager.GetHighlightingAttribute(highlighting);

  if (severity == Severity.INFO && attribute.OverlapResolve == OverlapResolveKind.NONE)
    return false;

  return true;
}
```

The default implementation, as shown above, looks up the configured severity of the highlight, and hides it if it's set to `Severity.INFO` and its overlap resolution is set to `OverlapResolveKind.NONE`. This combination is usually used by static highlights, identifier highlights, gutter marks and context highlighting, which are not usually useful in a highlighter test. However, if the test needs to see this kind of highlighting, simply override the `HighlightingPredicate` method to suit.

### Customising the base class

The following properties and methods can be overridden to alter the behaviour of the tests:

* `WithProject` is called once the project is created and just before the main test code is run. This is a useful hook for project specific setup, such as setting C# language level, or custom tools on added files.
* `ColorIdentifiers` - override this property to enable ReSharper's "Color identifiers" option, which will highlight different types of identifiers with different colours, e.g. classes get one colour, interfaces get another. This is set to `false` by default.
* `ProjectOutputType` can be overridden if the output type of the project (console, assembly, etc.) is relevant to the tests.
* `GetReferencedAssemblies` is used to add extra references to the created project. The default implementation will look for any attributes on the test class that implement `ITestLibraryReferencesProvider`, such as `TestReferencesAttribute`, or derived classes like `TestNetCore45Attribute`, and add any references, expanding environment variables.
* `GetSdkReferences` will add SDK references, by default looking for attributes that implement `ITestSdkReferencesProvider`, such as `TestSdksAttribute` or `TestWinJSApplicationAttribute`.
* `ProcessAllFiles` can be overridden in order to process all files in the project. By default, only one file is expected, and only the first found is run.
* `CreateHighlightingDumper` can be overridden to provide an instance of `TestHighlightingDumper` that outputs the highlights in a different format. A typical use of this is to replace the default implementation with an instance of `TestHighlightingDumperWithType`, which simply adds the type name of the highlight to the output.

## Derived classes

The following derived classes provide language specific features, such as adding appropriate assembly references and setting up the default file extension:

* **`CSharpHighlightingTestBase`** provides overrides for `DoNamedTest` and `DoNamedTest2` that allow specifying a C# language level. The `CSharpHighlightTestNet45Base`, `CSharpHighlightingTestNet4Base` and `WinRTHighlightingTestBase` derived classes set up .net 4.5, .net 4 and WinRT tests, respectively.
* **`CssHighlightingTestBase`** provides an override of `DoNamedTest` that takes a CSS language level.
* **`HtmlHighlightingTestBase`** is a base class for testing highlights in HTML and HTML-like languages.
    * **`AspHighlightingTestBase`** provides a base class for testing highlights in `.aspx` files. It offers a couple of extra methods, including `DoTestWithConfig` and `DoTestFileWithControl`.
* **`JavaScriptHighlightingTestBase`** - base class for JavaScript highlight tests. Sets up some default settings.
* **`TodoHighlightingTestBase`** provides a custom dumper to format todo items.
* **`TypeScriptHighlightingTestBase`** - base class for TypeScript highlights.
* **`VBHighlightingTestBase`** sets up the test for VB tests. Allows setting the language level before running a test:
    ```csharp
    [Test]
    public void Test01()
    {
      using (new LanguageLevelCookie(this, VBLanguageLevel.Vb10))
        DoNamedTest();
    }
    ```
* **`XamlHighlightingTestBase`** - abstract base class for XAML tests.

## Testing gutter marks and dead code

Gutter marks are essentially the same as normal highlights - the icons are displayed in the gutter of the text editor, but the highlight is still applied to a section of code such as a method declaration. Similarly, the highlight has text, and the text and location is listed in the gold file in exactly the same way as normal (underlined) highlights.

In order to make the tests a little easier, the test class can override the `HighlightPredicate` method and filter the `IHighlighting` instance to ensure it is of a known type for the gutter marks under test, e.g.:

```csharp
protected override bool HighlightingPredicate(IHighlighting highlighting, IContextBoundSettingsStore settingsStore)
{
  return (highlighting is InheritanceMarkOnGutter) || (highlighting is RecursionMarkOnGutter);
}
```

Similarly, dead code is also just a highlight, and the tests are exactly the same.
