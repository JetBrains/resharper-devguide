[//]: # (title: Testing)

Testing language registration is fairly quick and easy. It's mostly checking that the Component Model has loaded the types correctly. This means the tests need to derive from `BaseTest`, so that the environment and Component Model has been initialised, but a solution hasn't been created.

Testing the `IProjectFileLanguageService` and PSI `LanguageService` implementations is not so straightforward. It's better to test them indirectly, by testing other functionality such as lexing, parsing, and so on.

## Testing project file type definition

To test if a project file type has been registered correctly, you can query the file type from `IProjectFileTypes`, or check that the `INSTANCE` field has been set to a non-`null` value.

If `IProjectFileLanguageService` has been implemented, you can also check that `IProjectFileExtensions` returns the correct project file type for a given file extension.

It's also useful to include a test marked `[Explicit]`, that doesn't normally run, and will dump all of the known project file types for debugging purposes.

 >  It's likely that a project file type test is one of the first tests implemented in a test project. Make sure that the test project includes a proper assembly reference, so that the Component Model will load the components from the assembly under test.
 >
 {type="note"}

```csharp
[TestFixture]
public class MyProjectFileTypeTests : BaseTest
{
  [Test]
  public void ProjectFileTypeIsRegistered()
  {
    // This line ensures that if this is the first test, there is at least one
    // reference to the assembly under test. If there are no references, the
    // Component Model doesn't know to use the assembly under test. The
    // reference to MyProjectFileType.Name doesn't count, as it's a const.
    Assert.NotNull(MyProjectFileType.Instance);

    var projectFileTypes = Shell.Instance.GetComponent<IProjectFileTypes>();
    Assert.NotNull(projectFileTypes.GetFileType(MyProjectFileType.Name));
  }

  [Test]
  public void ProjectFileTypeFromExtension()
  {
    // This requires IProjectFileTypeLanguageService to be implemented, otherwise
    // ReSharper returns UnknownProjectFileType.INSTANCE
    var projectFileExtensions = Shell.Instance.GetComponent<IProjectFileExtensions>();
    Assert.AreSame(MyProjectFileType.Instance, projectFileExtensions.GetFileType(MyProjectFileType.MY_EXTENSION));
  }

  [Test, Explicit]
  public void DumpProjectFileTypes()
  {
    var projectFileTypes = Shell.Instance.GetComponent<IProjectFileTypes>();
    foreach (var projectFileType in projectFileTypes.All)
    {
      Console.WriteLine(projectFileType.PresentableName);
    }
  }
}
```

## Testing the PSI language service registration

To test the PSI language service, simply check the language's `Instance` static field is not null, and check that ReSharper's `Languages` class can retrieve the language by name.

Again, it's useful to have a test marked `[Explicit]` that will print all of the registered languages names for debugging.

```csharp
[TestFixture]
public class MyLanguageTests : BaseTest
{
  [Test]
  public void LanguageIsRegistered()
  {
    Assert.NotNull(MyLanguage.Instance);
    Assert.NotNull(Languages.Instance.GetLanguageByName(MyLanguage.Name));
  }

  [Test, Explicit]
  public void DumpLanguages()
  {
    foreach (var languageType in Languages.Instance.All)
    {
      Console.WriteLine(languageType.PresentableName);
    }
  }
}
```
