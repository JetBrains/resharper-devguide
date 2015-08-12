---
---

# Test Environment

ReSharper tests are functional tests, testing a complete end to end slice of ReSharper functionality, rather than a smaller unit such as a class. This is because it is impractical to try and mock an abstract syntax tree and a semantic model - it is simpler to build a real syntax tree and semantic model and exercise real functionality with the actual data model.

Under normal operation, ReSharper runs as an extension to Visual Studio, but this is also impractical for testing purposes. When running tests, ReSharper is started and runs as an in-memory test environment.

To create an in-memory ReSharper instance, ReSharper tests use an [NUnit setup fixture](http://nunit.org/index.php?p=setupFixture&r=2.6.3), which is a fixture class that is run once, before any tests that are in the same namespace or below are run. (E.g. the `Foo.MySetupFixture` setup fixture class is run before any tests in the `Foo` namespace or below, such as `Foo.Bar.MyTests`, are run. See the [NUnit docs for more details on setup fixtures](http://nunit.org/index.php?p=setupFixture&r=2.6.3)). This allows the fixture to set up and tear down any data required for those tests - i.e. the test environment.

An extension should create a class that derives from `ExtensionTestEnvironmentAssembly<TTestEnvironmentZone>` and mark it with the `[SetUpFixture]` attribute. The `TTestEnvironmentZone` type parameter is used by the test environment to decide what parts of the ReSharper platform are initialised, and available during testing. See [Zones](#zones) below for more details.

> **NOTE** The ReSharper 9.0 SDK did not ship with the `ExtensionTestEnvironmentAssembly` class, and testing on 9.0 is not officially supported. The recommended approach is to upgrade your plugins to 9.1 or later.

```csharp
namespace Foo.Tests
{
  // See Zones below
  [ZoneDefinition]
  public interface IMyTestZone : ITestsZone, IRequire<PsiFeatureTestZone>
  {
  }

  [SetUpFixture]
  public class TestEnvironment : ExtensionTestEnvironmentAssembly<IMyTestZone>
  {
  }
}
```

This is all that is necessary to create a ReSharper test environment based on the files in the output folder. The ReSharper test environment will scan all of the files in the output folder looking for components to load via the [Component Model](/Platform/ComponentModel.md), which means that the plugin does not need to be installed into this in-memory ReSharper instance (it still needs to be [installed into an interactive instance](/Extensions/Plugins/ProjectSetup/InitialInstallation.md) for manual testing).

## Zones

Zones are a feature that ReSharper uses to manage shipping multiple products with overlapping assemblies, features and licensing requirements. Essentially, they are used to decide what parts of the application are active. More details on the use of Zones for testing can be found in the [Testing Zones](Zones.md) section.

Briefly, an extension should create a test zone definition that states what zones are required in order to run the tests. Typically, the following is sufficient. The use of the `IRequire<PsiFeatureTestZone>` states that the test environment needs to run all components that have declared that they are required for testing PSI features, which includes code completion, unit testing, parsing, analysis and so on.

```csharp
[ZoneDefinition]
public interface IMyTestZone : ITestsZone, IRequire<PsiFeatureTestZone>
{
}

[SetUpFixture]
public class TestEnvironment : ExtensionTestEnvironmentAssembly<IMyTestZone>
{
}
```
