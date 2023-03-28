[//]: # (title: Testing)

 >  This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.
 >
 {type="warning"}



ReSharper's SDK provides infrastructure for testing your plugin.

But why do we need the test infrastructure? Well, if we're talking about isolated behavior of your plugins, ordinary unit tests do just fine. However, these tests assume that you mock the ReSharper infrastructure or that your code is well-isolated from it. But the interaction with the ReSharper infrastructure is best tested with specific test infrastructure. The infrastructure lets you, e.g., spin up a real instance of Visual Studio, open up a solution and, say, try out a particular context action.

The general rules for testing is that for particular types of plugin items, you get corresponding base classes. For example, to test Live Template Macros, the ReSharper SDK provides a base class called MacroTestBase. By inheriting this class, you can override its members in order to, e.g., specify input and expected output files. Please note that base classes also 'branch off' in terms of what functionality of a particular feature they test. For example, when testing a context action, you have a base class which tests the availability of a context action, and another base class for testing its actual effects.

## Test environment assembly

In order to tell ReSharper that the assembly contains tests that should be executed by the ReSharper infrastructure, the assembly must include a type decorated with the `SetUpFixture` attribute and inheriting from `ReSharperTestEnvironmentAssembly`.

NUnit will create any set up fixtures before running any tests defined in or below the namespace of the set up fixure class. For this reason, the project templates distributed with the ReSharper SDK create the set up fixture in the root namespace, and place it in the `AssemblyInfo.cs` file.

However, it is also useful to move the class to a child namespace, and ensure that all acceptance tests are declared in or below this child namespace. This allows for real unit tests to also be declared in the assembly, and not pay the cost of initialising a ReSharper environment when running unit tests.

```csharp
[SetUpFixture]
public class MyTestEnvironmentAssembly : ReSharperTestEnvironmentAssembly
{
  /// <summary>
  /// Gets the assemblies to load into test environment.
  /// Should include all assemblies which contain components.
  /// </summary>
  private static IEnumerable<Assembly> GetAssembliesToLoad()
  {
    // Test assembly
    yield return Assembly.GetExecutingAssembly();

    // Plugin code - TypeUnderTest is implemented in the plugin assembly
    yield return typeof(TypeUnderTest).Assembly;
  }

  public override void SetUp()
  {
    base.SetUp();
    ReentrancyGuard.Current.Execute("LoadAssemblies", () =>
    {
      Shell.Instance.GetComponent<AssemblyManager>().LoadAssemblies(GetType().Name, GetAssembliesToLoad());
    });
  }

  public override void TearDown()
  {
    ReentrancyGuard.Current.Execute("UnloadAssemblies", () => 
    {
      Shell.Instance.GetComponent<AssemblyManager>().UnloadAssemblies(GetType().Name, GetAssembliesToLoad());
    });
    base.TearDown();
  }
}
```

You can copy the above code verbatim into your application; just be sure to get the `AssemblyManager` to load the assemblies containing the types you plan to test. Failure to do so will cause the infrastructure to run tests against your plugin without actually loading the plugin assemblies into the component model.

## Input Files

If you are writing a plugin which modifies code, a typical test would involve two files - an initial input file and an expected output file. The output file is a file which represents the expected output state after a particular action has executed.

Input and output files involve special symbols and commands for indicating the state of the file. For example, by writing `{caret}`, you indicate where the caret is located.

ReSharper adopts a convention-based approach to locating your tests. The convention suggests the following structure for the projects and the associated tests:

```
\src
  \your_project.sln
  \your_project_files
\test
  \data
    \your_feature_name
      \availability01.cs
      \execute01.html
      \execute01.html.gold
```

In the above structure, in order to get your feature files to be picked up by the test runner, you need to do the following:

* Create a folder called `your_feature_name`
* In your test, the value of the overridden `RelativeTestDataPath` property should correspondingly be `your_feature_name`
* Your test cases should invoke the appropriate test files. For example, if your test data file is named `availability01.cs`, your test can appear as follows:

    ```csharp
    [TestCase("availability01")]
    [Test]
    public void TestExecution01(string testSrc)
    {
      DoOneTest(testSrc);
    }
    ```

## A Note on File Extensions

Since tests typically default to searching for files with `.cs` extensions, if you use a different extension you'll need to tell ReSharper about it. The way to do it is by decorating the test fixture with the `[TestFileExtension]` attribute. You can either provide it a string of the file extension (e.g., `".html"`) or reference one of the existing string constants that are kept in the project file types, for example `HtmlProjectFileType.HTML_EXTENSION`.

## Test Data Mark-up

When working with test data, you can use special symbols (delimited in curly braces) in order to specify various test-related settings. These are:

* `{caret}` - indicates the position of the caret
* `{selstart}` and `{selend}` - indicate the start and end position of selection
* `{sourceN}` and `{targetN}` - indicate the Nth movement of the caret. the source indicates where the caret is moving from, the target where it is moving to.

In addition to the above, you can set up the notation of `${settingname:<value>`} in your test data. You can subsequently use the `GetSetting()` method to read the value of the setting with the specified name in your test.

## Troubleshooting

Plugin tests may fail for the following reasons:

* Your component has a dependency on a Visual Studio object, such as an `IVsShell`. In this case you have two options:
    * You can either move this component to a separate project that is not referenced by your tests; or
    * You can edit the attribute of the component, specifying `ProgramConfigurations = VS_ADDIN`. This ensures the component doesn't get loaded in tests.
* You are attempting to run tests on a plugin while that plugin is already installed in Visual Studio. When testing plugins, ReSharper takes into account all the plugins currently installed, which may cause conflicts with the plugin under test.
* The above may also be an issue with third-party plugins where plugin developers have forgotten to decorate VS-reliant interfaces with appropriate attributes to prevent them from being loaded in tests.
