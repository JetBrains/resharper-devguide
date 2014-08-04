# Testing External Annotations

There are a couple of ways of testing External Annotations. ReSharper provides the `ExternalAnnotationsTestBase` class which has a couple of methods for testing that certain annotations are applied to various types and type members. Unfortunately, it doesn't provide methods to test all annotation types, although more support can be easily implemented.

An alternative method of testing annotations is to verify the highlights and warnings that ReSharper produces when analysing code that uses the code elements being annotated. For example, if a code element is marked with the `[NotNull]` attribute, check that ReSharper produces a warning for a null check.

## Loading the annotations

Before any tests can run, the annotations need to be loaded by the test environment. ReSharper doesn't do this by default, it only loads annotations that are installed with the product, or as extensions. Instead, they have to be loaded explicitly by the test assembly, by implementing `IExternalAnnotationsFileProvider` and marking the component with the `[ShellComponent]` attribute.

The following class will load all annotations stored in a specific folder, which, by default, is `test\data\annotations`, but by replacing the implementation of `RelativeTestDataPath` you can change it to another location relative to `test\data`, even outside of the `test\data` folder structure. Or you can completely replace `BaseTestDataPath` to return any location you want.

This provider will load all XML files that are immediate children of the file location. These are expected to be named after the assembly, e.g. `myAssembly.dll` would need a `myAssembly.xml` file in the root location. It will also recursively look for any XML files in child directories. Either the directory should be named after the assembly, in which case all files in that directory apply to the assembly, or the file should be named after the assembly. This is the same folder structure and naming scheme used when annotations are distributed inside an extension package.

```cs
[ShellComponent]
public class TestExternalAnnotationsProvider : IExternalAnnotationsFileProvider
{
  private readonly OneToSetMap<string, FileSystemPath> annotations;
  private FileSystemPath cachedBaseTestDataPath;

  public TestExternalAnnotationsProvider()
  {
    var location = TestDataPath2;
    annotations = new OneToSetMap<string, FileSystemPath>(StringComparer.OrdinalIgnoreCase);

    foreach (var file in location.GetChildFiles("*.xml"))
      annotations.Add(file.NameWithoutExtension, file);
    foreach (var directory in location.GetChildDirectories())
    {
      foreach (var file in directory.GetChildFiles("*.xml", PathSearchFlags.RecurseIntoSubdirectories))
      {
        annotations.Add(file.NameWithoutExtension, file);
        annotations.Add(file.Directory.Name, file);
      }
    }
  }

  public IEnumerable<FileSystemPath> GetAnnotationsFiles(AssemblyNameInfo assemblyName = null, FileSystemPath assemblyLocation = null)
  {
    if (assemblyName == null)
      return annotations.Values;
    return annotations[assemblyName.Name];
  }

  private FileSystemPath BaseTestDataPath
  {
    get
    {
      if (cachedBaseTestDataPath == null)
        cachedBaseTestDataPath = TestUtil.GetTestDataPathBase(GetType().Assembly);
      return cachedBaseTestDataPath;
    }
  }

  private string RelativeTestDataPath
  {
    // This can be relative, e.g. @"..\..\ExternalAnnotations"
    get { return @"annotations"; }
  }

  private FileSystemPath TestDataPath2
  {
    get { return BaseTestDataPath.Combine(RelativeTestDataPath); }
  }
}
```

## Targetting the assembly

External annotations don't target source files, but already existing, compiled assemblies. In order to test that annotations are correctly applied, the in-memory, temporary project created by the test environment needs to add references to the target assemblies.

This is easily done using the `[TestReferences]` attribute on the test class. The parameters to the attribute are the names of the assemblies to reference. The values can be a filename, or a relative or absolute file path. If the value is a filename or a relative file path, the actual path is resolved against the `TestDataPath2` property (which resolves to `test\data` plus the value of `RelativeTestDataPath`, which in the case of `ExternalAnnotationsBase` is `string.Empty`). If the value is an absolute file path, it is used as-is.

```cs
[TestReferences("myAssembly.dll", "mySupport.dll")]
public class MyAssemblyAnnotationsTest : ExternalAnnotationsTestBase
{
  // ...
}
```

The referenced assembly names can also contain environment variable names, which will be expanded before use. This can be very useful, as environment variables can be set programmatically by overriding the `SetUp` method. E.g.:

```cs
[TestReferences(@"%MY_ASSEMBLY_PATH%\myAssembly.dll")]
public class MyAssemblyAnnotationsTest : ExternalAnnotationsBase
{
  public override void SetUp()
  {
    base.SetUp();

    Environment.SetEnvironmentVariable("MY_ASSEMBLY_PATH", BaseTestDataPath.Combine("lib"));
  }
}
```

If you need more control over the references you're adding, you can override the `BaseTestWithSingleProject.GetReferencedAssemblies` method.

> **Note** Remember that external annotations can target multiple versions of an assembly by specifying the version in the assembly `name` attribute in the XML file. To test multiple versions, simply create multiple test classes, specifying different references, perhaps with different paths. If the multiple versions share test cases, they can implemented in a base class and shared via inheritance.

## Using the base class

The `ExternalAnnotationsTestBase` class provides several methods that will assert that a given type, type member or parameter has a particular annotation attribute applied. Each method must identify which type or type member the annotations is expected for, and this is done using the type or type member's XML Doc ID, for example, `"M:System.IO.Path.GetTempPath"` or `"P:System.Collections.CollectionBase.List"`. The remaining parameters specify the parameter name, if required, and the condition to assert.

> **Note** The prefix in the name of the XML Doc ID identifies the type of the code element - `T` for type, `M` for method, `P` for property and so on. See the [MSDN documentation on XML Doc IDs](http://msdn.microsoft.com/en-us/library/fsbx0t7x.aspx) for more details.

The methods defined are:

* **`AssertFormatMethod`** - asserts that the given method is annotated with the `[StringFormatMethod]` attribute, and the correct parameter name is specified.
* **`AssertMember`** - asserts that the given member (i.e. method, property, field, etc.) is annotated with either the `[NotNull]` or `[CanBeNull]` attributes.
* **`AssertParameter`** - asserts that the given member's parameter is annotated with either `[NotNull]` or `[CanBeNull]`.

As can be seen, these methods do not cover all of the annotations, but it is easy to add support for more, as shown below.

### Extending the base class

The base class doesn't provide many asserts by default, but it is easy to add support for more. The basic structure of the assert methods are very similar, and can be abstracted out to a helper method:

```cs
private void AssertHelper(string xmlDocId, Action<CodeAnnotationsCache, IDeclaredElement> assert)
{
  // We're not checking source files, but referenced assemblies
  var sourceFiles = EmptyList<string>InstanceList;
  WithSingleProject(sourceFiles, (lifetime, solution, project) =>
  {
    var psiModule = solution.PsiModules().GetPrimaryPsiModule(project);
    var psiServices = solution.GetPsiServices();

    // Get the IDeclaredElement from the XML Doc ID
    var declaredElement = XMLDocUtil.ResolveId(psiServices, xmlDocId, psiModule, true, project.GetResolveContext();
    Assert.NotNull(declaredElement, "Declared element cannot be resolved from XMLDocID {0}", xmlDocId);

    var annotationsCache = psiServices.GetAnnotationsCache();

    assert(annotationsCache, declaredElement);
  });
}
```

This helper method runs the test with a solution that contains a single project without any source files (remember that external annotations only apply to referenced, compiled assemblies, so we don't need any source files). It then uses the `XMLDocUtil.ResolveId` method to get an `IDeclaredElement` for the type or type member passed in to the method, which is then passed to the given `assert` action delegate, together with an instance of `CodeAnnotationsCache`.

Using the helper method is straightforward. For example, to check the assert condition applied to a parameter:

```cs
protected void AssertParameterAssertCondition(string xmlDocId, string parameterName, AssertConditionType? conditionType)
{
  AssertHelper(xmlDocId, (cache, element) =>
  {
    var parametersOwner = element as IParametersOwner;
    Assert.NotNull(parametersOwner, "Declared element is not an IParametersOwner {0}", xmlDocId);

    var parameter = parametersOwner.Parameters.SingleOrDefault(p => p.ShortName == parameterName);
    Assert.NotNull(parameter, "Parameter \"{0}\" is not found on {1}", parameterName, xmlDocId);

    var actual = cache.GetParameterAssertionCondition(parameter);
    Assert.AreEqual(conditionType, actual);
  });
}
```

This retrieves the parameter from the given `IDeclaredElement`, then uses the passed in `CodeAnnotationsCache` to retrieve the assert condition for the parameter, and assert that it is what is expected.

## Testing warnings and highlights

Instead of testing the annotations are applied to members, you can also test the highlights generated (or not) in response to the annotations, by deriving from `HighlightingTestBase` or one of its derived types, such as `CSharpHighlightingTestBase`.

This is a simple highlighting test, which analyses a source file and creates a temporary file that contains the text of the original file, plus a list of all highlights that are applied (if any). If the analysis is affected by external annotations, such as a `[NotNull]` annotation, extra highlights can be applied to the code that wouldn't be applied if the annotations weren't there. The temporary file, with the list of highlights, is compared to a previously stored 'gold' file. If it differs, the test fails.

The tests can be run with the `DoOneTest` method, where the name of the source file is passed in, or by using the `DoNamedTest2` method, which takes the file name from the test method, minus any 'test' prefix.

```cs
[Test]
public void Should_show_annotations()
{
  // Looks for 'MyAnnotations.cs'. The '.cs' is added automatically
  DoOneTest("MyAnnotations");
}

[Test]
public void TestMyAnnotations()
{
  DoNamedTest2(); // Looks for 'MyAnnotations.cs'
}
```

You might prefer to use this method over checking the annotations are applied as it checks the outcome of the annotation, rather than just that the annotation is applied.

