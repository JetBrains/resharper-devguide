---
---

# Testing External Annotations

There are two approaches to testing External Annotations. The first is to use the `CodeAnnotationsCache` class to assert that annotations are applied to specific code elements in the compiled assemblies. The second is to assert the highlights and warnings that are affected by annotations. For example, an element marked with the `[NotNull]` attribute will cause ReSharper to highlight a null check as unnecessary.

Which method you choose is up to you. The first approach tests that the external annotations are correctly applied, while the second asserts the expected result of applying the annotations. ReSharper doesn't prescribe a specific approach; there are no base classes or helper methods specifically for testing external annotations. The first approach requires some helper methods, and an example base class is provided below, while the second approach is a normal highlighting test.

## Loading the annotations

Before any tests can run, the annotations need to be loaded by the test environment. ReSharper doesn't do this by default, it only loads annotations that are installed with the product, or as extensions. Instead, they have to be loaded explicitly by the test assembly, by implementing `IExternalAnnotationsFileProvider` and marking the component with the `[ShellComponent]` attribute.

The following class will load all annotations stored in a specific folder, which, in this example will be `test\data\annotations`, but by replacing the implementation of `RelativeTestDataPath` you can change it to another location relative to `test\data`, even outside of the `test\data` folder structure. Or you can completely replace `BaseTestDataPath` to return any location you want.

This provider will load all XML files that are immediate children of the file location. These are expected to be named after the assembly, e.g. `myAssembly.dll` would need a `myAssembly.xml` file in the root location. It will also recursively look for any XML files in child directories. Either the directory should be named after the assembly, in which case all files in that directory apply to the assembly, or the file should be named after the assembly. This is the same folder structure and naming scheme used when annotations are distributed inside an extension package.

```csharp
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

## Targeting the assembly

External annotations don't target source files, but already existing, compiled assemblies. In order to test that annotations are correctly applied, the in-memory, temporary project created by the test environment needs to add references to the target assembly/assemblies.

This is easily done using the `[TestReferences]` attribute on the test class. The parameters to the attribute are the names of the assemblies to reference. The values can be a filename, or a relative or absolute file path. If the value is a filename or a relative file path, the actual path is resolved against the `TestDataPath2` property (which resolves to `test\data` plus the value of `RelativeTestDataPath`). If the value is an absolute file path, it is used as-is.

```csharp
[TestReferences("myAssembly.dll", "mySupport.dll")]
public class MyAssemblyAnnotationsTest : ExternalAnntotionsTestBase2
{
  // ...
}
```

The referenced assembly names can also contain environment variable names, which will be expanded before use. This can be very useful, as environment variables can be set programmatically by overriding the `SetUp` method. E.g.:

```csharp
[TestReferences(@"%MY_ASSEMBLY_PATH%\myAssembly.dll")]
public class MyAssemblyAnnotationsTest : ExternalAnnotationsTestBase2
{
  public override void SetUp()
  {
    base.SetUp();

    Environment.SetEnvironmentVariable("MY_ASSEMBLY_PATH", BaseTestDataPath.Combine("lib"));
  }
}
```

If you need more control over the references you're adding, you can override the `BaseTestWithSingleProject.GetReferencedAssemblies` method.

> **NOTE** Remember that external annotations can target multiple versions of an assembly by specifying the version in the assembly `name` attribute in the XML file. To test multiple versions, simply create multiple test classes, each referencing a different version of the target assembly. If the multiple versions share test cases, they can implemented in a base class and shared via inheritance.
>
> If using base classes, the `TestReferencesAttribute.Inherits` property declares whether `TestReferences` attributes on the base class are used or not. By default, they aren't - the `TestReferences` attribute of declared classes override the base class.

## Testing applied annotations

The following code snippet defines a class called `ExternalAnnotationsTestBase2` which provides a helper method to convert an XML Doc ID to an `IDeclaredElement`. It then gets an instance of `CodeAnnotationsCache` and both are passed to the assert method, which can use the cache to ask for the annotations applied to the given code element.

> **NOTE** ReSharper already defines a class called `ExternalAnnotationsTestBase` that provides similar functionality used internally for testing external annotations. However, it is unsuitable to use as a base class for plugin tests, as it defines tests for the standard BCL annotations, which would also run for your plugin tests.

The base class looks like this:

```csharp
public abstract class ExternalAnnotationsTestBase2 : BaseTestWithSingleProject
{
  protected override RelativeTestDataPath { get { return string.Empty; } }

  private void AssertHelper(string xmlDocId, Action<CodeAnnotationsCache, IDeclaredElement> assert)
  {
    // We're not checking source files, but referenced assemblies
    var sourceFiles = EmptyList<string>InstanceList;

    WithSingleProject(sourceFiles, (lifetime, solution, project) =>
    {
      RunGuarded(() =>
      {
        var psiModule = solution.PsiModules().GetPrimaryPsiModule(project);
        var psiServices = solution.GetPsiServices();

        // Get the IDeclaredElement from the XML Doc ID
        var declaredElement = XMLDocUtil.ResolveId(psiServices, xmlDocId, psiModule, true, project.GetResolveContext();
        Assert.NotNull(declaredElement, "Declared element cannot be resolved from XMLDocID {0}", xmlDocId);

        var annotationsCache = psiServices.GetAnnotationsCache();

        assert(annotationsCache, declaredElement);
      });
    });
  }

  protected void AssertMeansImplicitUseAttribute(string xmlDocId)
  {
    AssertHelper(xmlDocId, (cache, element) =>
    {
      var attributesOwner = element as IAttributesOwner;
      Assert.NotNull(attributesOwner, "Declared element is not an IAttributesOwner {0}", xmlDocId);

      var attributeInstances = attributesOwner.GetAttributeInstances(false);
      foreach (var attributeInstance in attributeInstances)
      {
        ImplicitUseKindFlags kindFlags;
        ImplicitUseTargetFlags targetFlags;
        if (cache.IsMeansImplicitUse(attributeInstance, out kindFlags, out targetFlags))
          return;
      }

      Assert.Fail("Declared element is not marked as implicit use {0}", xmlDocId);
    });
  }

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
}
```

Other assert methods can be added similar to `AssertParameterAssertCondition` that also call into `CodeAnnotationsCache` in order to assert annotations are applied.

> **NOTE** The prefix in the name of the XML Doc ID identifies the type of the code element - `T` for type, `M` for method, `P` for property and so on. See the [MSDN documentation on XML Doc IDs](http://msdn.microsoft.com/en-us/library/fsbx0t7x.aspx) for more details.

<!-- Comment to separate the notes -->

> **WARNING** This approach requires that ReSharper can resolve the types and constructors of the `JetBrains.Annotations` attributes referenced in the external annotations file. If ReSharper cannot resolve these types, the annotations are not applied, and tests can fail.
>
> ReSharper can automatically resolve types against the `JetBrains.Annotations.dll` assembly in the product install directory (by using a custom `IPsiModule`). In normal usage, this effectively means all projects can resolve references to the annotation attributes.
>
> However, in tests, the product directory is the location of the test assembly, and the `JetBrains.Annotations.dll` assembly is not referenced in a test project, and therefore not copied to the `bin\Debug` folder. (It is not referenced simply because ReSharper itself doesn't include any binary references to the `.dll` to prevent versioning conflicts when multiple products are installed.)
>
> This can be fixed in a couple of ways. Firstly, add a reference to `JetBrains.Annotations.dll` in your plugin test project, or a custom post-build step that copies `$(ReSharperSdkBinaries)\JetBrains.Annotations.dll` to the output folder.
>
> Secondly, add `JetBrains.Annotations.dll` to the `[TestReferences]` attribute, and place a copy in the appropriate `test\data` location (`TestDataPath2` + `RelativeTestDataPath`).
>
> Alternatively, add a C# source file to the temporary, in-memory test project that contains the source of the annotation attributes. The file will live in the location represented by `TestDataPath2` and `RelativeTestDataPath` (e.g. `"test\data\"` + `RelativeTestDataPath`). The contents of the file come from ReSharper's options dialog (ReSharper &raquo; Options &raquo; Code Inspection &raquo; Code Annotations &raquo; Copy default implementation to clipboard).
>
> This approach works well, except the source file doesn't contain all annotations - older, obsolete annotations are not included (for example, `AssertionConditionAttribute`, while still supported, is not included in the default implementation, as it has been replaced by the more versatile `ContractAnnotationAttribute`). If you require the older annotations, add a binary reference.

## Testing warnings and highlights

Instead of testing that the annotations are applied to members, you can also test the highlights generated (or not) in response to the annotations, by deriving from `HighlightingTestBase` or one of its derived types, such as `CSharpHighlightingTestBase`.

This is a simple highlighting test, which analyses a source file and creates a temporary file that contains the text of the original file, plus a list of all highlights that are applied (if any). If the analysis is affected by external annotations, such as a `[NotNull]` annotation, extra highlights can be applied to the code that wouldn't be applied if the annotations weren't there. The temporary file, with the list of highlights, is compared to a previously stored 'gold' file. If it differs, the test fails.

The tests can be run with the `DoOneTest` method, where the name of the source file is passed in, or by using the `DoNamedTest2` method, which takes the file name from the test method, minus any 'test' prefix.

```csharp
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

You might prefer to use this method over checking the annotations are applied as it checks the outcome of the annotation, rather than just that the annotation is applied. While this approach initially seems easier, it should be noted that each test requires a new source file for each scenario. The first approach, while requiring extra initial set up for creating the base class and importing `JetBrains.Annotations.dll`, makes each test a simple call to a helper method to assert the annotation. This might be easier if you have many annotations you wish to test.

