---
---

# Zones

> **WARNING** This topic assumes familiarity with the concept and usage of both the [Component Model](/Platform/ComponentModel.md) and [Zones](/Platform/Zones.md) topics.

The [Component Model](/Platform/ComponentModel.md) is responsible for bootstrapping the ReSharper environment, creating the containers and components (classes) that provide ReSharper functionality. To work with the constraints of shipping multiple products with overlapping and complementary feature sets, assemblies, and licensing requirements, the Component Model uses [Zones](/Platform/Zones.md) to decide what components should be activated. It is strongly recommended that you have the topics on the [Component Model](/Platform/ComponentModel.md) and [Zones](/Platform/Zones.md) before continuing.

The same process happens when creating the in-memory [test environment](TestEnvironment.md). The Component Model is still responsible for creating the containers and components, and still uses Zones to filter what components are activated. However, the process for deciding what zones are active is different.

During normal startup, the host provides a set of zones that are already active. These zones provide details about the current host, and a zone can "require" a host zone to allow for activation only in a specific host. (E.g. a zone can require `IVisualStudioZone` to only be active when hosted inside Visual Studio.)

When creating the test environment, the `ExtensionTestEnvironmentAssembly<TTestEnvironmentZone>` derived class has to specify a type parameter that derives from `ITestsZone`, which is a host zone definition that indicates that the current host is for running tests.

> **TIP** `ITestsZone` should not be confused with `IUnitTestingZone`, which is the zone that provides the unit testing feature set.

Any zone definitions that the derived `ITestsZone` definition requires will be activated by the test environment. For example:

```csharp
[ZoneDefinition]
public interface IMyTestsZone : ITestsZone, IRequire<PsiFeaturesTestZone>
{
}

[SetUpFixture]
public class TestEnvironment : ExtensionTestEnvironmentAssembly<IMyTestsZone>
{
}
```

This will create a test environment that activates `IMyTestsZone` (and therefore `ITestsZone` through inheritance). It will also activate any zones it requires - in this case, `PsiFeaturesTestZone`, which is a convenience definition that includes most zones required for testing the majority of ReSharper functionality.

```csharp
[ZoneDefinition]
public class PsiFeatureTestZone : IZone,
  IRequire<DaemonZone>, IRequire<NavigationZone>, IRequire<ILanguageJavaScriptZone>,
  IRequire<ILanguageCSharpZone>, IRequire<ILanguageCssZone>, IRequire<ILanguageVBZone>,
  IRequire<ILanguageMsBuild>, IRequire<ILanguageXamlZone>, IRequire<ILanguageNAntZone>,
  IRequire<ILanguageResxZone>, IRequire<ILanguageRegExpZone>, IRequire<ILanguageAspZone>,
  IRequire<ILanguageRazorZone>, IRequire<ILanguageIlZone>, IRequire<ICodeEditingZone>,
  IRequire<IInternalVisibilityZone>, IRequire<ExternalSourcesZone>, IRequire<IUnitTestingZone>,
  IRequire<IMsTestUntilVs09Zone>, IRequire<SymbolsImplZone>, IRequire<SweaZone>,
  IRequire<ITestsZone>, IRequire<IParameterInfoZone>, IRequire<ICodeEditingOptionsPageImplZone>
{
}
```

This is a good zone definition for tests to require by default, as it activates a lot of other zones that will be useful for testing, such as `DaemonZone` and `SweaZone` for highlighting and analysis, `ICodeEditingZone` for code completion and `IUnitTestingZone` for unit testing. It also includes a lot of language support, although it doesn't include the C++ language. 

It is possible to require a subset of these zones. If a plugin's tests do not need JavaScript support, or unit testing support, they can be omitted, by requiring only what is needed as `IRequire<TZone>` interfaces on the `ITestsZone` derived zone definition. E.g.:

```csharp
public interface IMyTestsZone : ITestsZone, IRequire<DaemonZone>, IRequire<ILanguageCppZone>
{
}
```

The custom `ITestsZone` zone definition can also be used to activate components that are defined in the test assembly, and that wouldn't normally be defined. See the section on [`Test Components`](TestComponents.md) for more details.
