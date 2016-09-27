---
---

# .NET Core Tests

> **NOTE** This topic discusses functionality added in ReSharper 2016.3

ReSharper's support for running tests for .NET Core projects is different to how it supports desktop CLR projects. Instead of using its own `JetBrains.ReSharper.TaskRunner.CLR4.exe` external process to run tests, it uses the .NET Core SDK `dotnet test` command line tool. The `dotnet test` command runs an external process from a NuGet package referenced in the project file (e.g. `dotnet-test-xunit` and `dotnet-test-xunit.exe`) which in turn runs your tests. But it can also be invoked in "design mode", which allows a host to connect to the running process and issue commands and receive messages. An IDE can use [this protocol](https://github.com/dotnet/core-docs/blob/master/docs/core/tools/test-protocol.md) to discover tests and run subsets of tests, getting feedback in real time.

ReSharper uses this protocol to run .NET Core tests. The workflow is as follows:

* *Discover tests in-editor*. The existing providers analyse source code in the same way they do for desktop CLR projects. This allows ReSharper to show the usual rich editing features - such as icons in the editor gutter, <kbd>Alt</kbd>+<kbd>Enter</kbd> actions for run and debug, and identifying methods and classes as tests in Find Usages.
* *Discover tests at build time*. When the project is built and the output assembly is changed, invoke `dotnet test` in design time mode to run test discovery. The list of tests discovered are treated as "the truth".
* *Map `dotnet test` IDs to ReSharper elements*. When a test is discovered, it is reported with an ID that is relevant only to the test runner from the NuGet package. A ReSharper test provider must map between this ID and a ReSharper unit test element.
* *Run or debug tests*. ReSharper currently only supports run and debug actions for .NET Core tests. When run or debug is invoked, `dotnet test` is invoked in design time mode, and the currently selected tests are run, by using the element map to produce a list of `dotnet test` IDs. Messages are received from the test process, and the ID to element map is again used to update the test status of the elements in the ReSharper UI.

Furthermore, a .NET Core project can support targeting multiple frameworks, such as .NET Core and/or the .NET Framework. ReSharper will run discovery for each target framework in a project, and create an `IUnitTestElement` for each test in each target framework. (ReSharper must create multiple test elements as each target framework build might have different `#if` defines that comment out tests.)

Adding support to an existing test provider for ReSharper is straightforward. ReSharper does all of the work of starting test discovery, running the tests and reporting test progress. A test provider only needs to map between `dotnet test` IDs and `IUnitTestElement` instances, which it can do by implementing a solution component that derives from `DotNetTestArtefactsExplorer`.

## Supporting .NET Core tests

ReSharper makes no assumptions about what types of projects a unit test provider supports. It is up to the provider to tell ReSharper when a project isn't supported, by handling the `IUnitTestProvider.IsSupported` method. If a provider returns `true` from one of these overloads, this provider will be called to discover tests in the editor.

> **NOTE** This also means that if a provider does NOT support .NET Core, its should return `false` for `IsSupported` for .NET Core projects. Failing to do this will mean that tests are still discovered in the editor, but running tests will have no effect, as there will be nothing to map between IDs and elements.

A provider should implement `IUnitTestProvider.IsSupported` so that the overloads look something like:

```csharp
public bool IsSupported(IProject project)
{
  var isReferencing = false;
  using (ReadLockCookie.Create())
  {
    // Are any of the project's target framework IDs referencing xunit or xunit.core?
    foreach (var targetFrameworkId in project.TargetFrameworkIds)
    {
      isReferencing |= IsReferencingAssembly(project, ourXunitCoreReferenceName, targetFrameworkId)
                        || IsReferencingAssembly(project, ourXunitReferenceName, targetFrameworkId);
    }
  }

  return isReferencing;
}

public bool IsSupported(IHostProvider hostProvider, IProject project)
{
  var supported = IsSupported(project);

  // If .NET Core, only support run and debug
  if (supported && project.IsProjectK())
  {
    return hostProvider.ID == WellKnownHostProvidersIds.DebugProviderId ||
            hostProvider.ID == WellKnownHostProvidersIds.RunProviderId;
  }
  return supported;
}
```

> **TODO** Show IsReferencingAssembly

Here, the provider is doing several things:

* Checking to see if the test assembly is referenced. If it isn't, there can't be any tests in the project, so the project isn't supported.
* Checking each target framework for the reference. A desktop CLR project will only have one target framework ID, but a .NET Core project can have many (e.g. .NET Core, .NET Framework, etc.). It is more than possible that one target framework includes the reference, but the other(s) don't.
* Checking to see if `IHostProvider.ID` is run or debug, if the project is .NET Core. ReSharper currently only supports running and debugging .NET Core tests, and doesn't yet support coverage, etc. Returning `false` will prevent ReSharper from trying to run the tests with coverage, for example.

> **NOTE** The provider is using `project.IsProjectK()` to check if the project is a .NET Core project. This is an extension method that checks to see if the project is a .NET Core project, and refers to .NET Core's initial code name of "project K".

## Mapping .NET Core tests to ReSharper elements

ReSharper provides the infrastructure for discovering and running .NET Core tests. However, `dotnet test` reports tests using an internal, opaque ID, relevant only to the test framework. In order to be able to work with ReSharper, a test extension must provide a mapping between this ID and ReSharper's own test elements.

A test provider needs to create a solution component that derives from `DotNetTestArtefactsExplorer` and overrides `DoDiscover` in order to provide this map.

The `DotNetTestArtefactsExplorer` base class will run and manage `dotnet test` in design mode for discovery. It implements `IUnitTestExplorerFromArtefacts`, and is called by ReSharper when a project is built and the output assemblies are changed. ReSharper will check `IUnitTestExplorerFromArtefacts.Provider.IsSupported(project)` method to see if the provider supports this project type, and also the `IUnitTestExplorerFromArtefacts.IsSupported(project)` method to see if this instance of the artefacts explorer supports the project. This allows for only running those artefacts explorer that are valid for the current project - where the test framework is supported, referenced and the explorer is the correct explorer for the project type.

The derived class should pass all required parameters into the base constructor, and should also implement the `DoDiscover` method:

```csharp
private readonly XunitServiceProvider myServices;

protected override void DoDiscover(DotNetTestDiscoverer discoverer, DotNetTestIdToElementMap testIdMap,
  IProject project, IUnitTestElementsObserver observer, CancellationToken token)
{
  var elementFactory = new UnitTestElementFactory(myServices, observer.OnUnitTestElementChanged);
  discoverer.Discover(project, testIdMap, observer, token,
    (p, o, t) => MapUnitTestElement(p, elementFactory, o.TargetFrameworkId, t));
}
```

The implementation is fairly straightforward, just call `discoverer.Discover` on the passed `DotNetDiscoverer` instance. It requires some information, which can be passed in from the parameters, and it also requires a mapping function. The function here is passed as a lambda, as it requires access to a factory class that will create instances of `IUnitTestElement` for elements that have not been discovered via in-editor discovery. This factory class likely already exists as part of the existing test provider implementation (note that `UnitTestElementFactory` shown above is the xUnit.net specific implementation).

The mapping function is passed three items, an `IProject`, the current `TargetFrameworkID` and the .NET Core `Test` object that describes the discovered test. Note that the `DoDiscover` method is also passed an instance of `DotNetTestIdToElementMap`. This is the component that maintains the mapping between .NET Core ID and `IUnitTestElement`. The mapper does not need to worry about this, as it is passed directly to the `Discover` method.

The mapping function is very much dependent on how the test framework generates its ID. It needs to pull information out of the ID, or use other details from the `Test` instance to identify test elements. The `Test.FullyQualifiedName` is the ID of the test, and depending on the framework, might be a full typename and method name that can be parsed to provide enough information (together with the `TargetFrameworkId`) to query for an existing element.

```csharp
private IUnitTestElement MapUnitTestElement(IProject project, UnitTestElementFactory elementFactory, TargetFrameworkId targetFrameworkId, Test test)
{
  using (ReadLockCookie.Create())
  {
    // This works with theories and methods that have already been discovered in-editor
    var element = myServices.GetElementById(project, targetFrameworkId, test.FullyQualifiedName);
    if (element != null)
    {
      UpdateCategories(element, test.Properties);
      return element;
    }

    var theoryName = RowName(test);
    var methodName = MethodName(test);
    var typeName = TypeName(test);

    if (string.IsNullOrEmpty(typeName))
      return null;

    var traits = GetTraits(test.Properties);

    var clrTypeName = new ClrTypeName(typeName);
    var typeElement = elementFactory.GetOrCreateTestClass(project, targetFrameworkId, clrTypeName, project.GetOutputFilePath(targetFrameworkId).FullPath, traits);
    if (typeElement == null)
      return null;

    if (string.IsNullOrEmpty(methodName))
      return typeElement;

    var shortName = methodName.Replace(typeName + ".", string.Empty);
    var methodElement = elementFactory.GetOrCreateTestMethod(methodName, project, targetFrameworkId, typeElement, clrTypeName, shortName,
      string.Empty, traits, false);
    if (methodElement == null)
      return null;

    if (string.IsNullOrEmpty(theoryName))
      return methodElement;

    return elementFactory.GetOrCreateTestTheory(theoryName, project, targetFrameworkId, methodElement,
      test.DisplayName);
  }
}
```

The example given above is the implementation for xUnit.net, which uses a fully qualified type name and method name for `Test.FullyQualifiedName`. The outline is as follows:

* Take the read lock with `ReadLockCookie.Create()`. This is required as we are looking up unit test elements in caches that can change.
* Parse `Test.FullyQualifiedName` into class name, method name and theory (row test) name. In this example, it is simple parsing - parsing a fully qualified method name by looking for parentheses and the last `.` in a name. See below for more details about row tests. Other frameworks might encode this ID differently.
* Find or create the element relating to the class name. It might need creating if the in-editor discovery hasn't found a similar test class.
* If the method name is empty, return the class element - this is the requested element.
* Find or create the element relating to the method name. The class element is passed in to act as a parent if the element needs creating.
* If the theory details are empty, return the method element.
* Find or create the element relating to the theory, and return it.

There are a couple of interesting points here:

* Return `null` if the element can't be found.
* Xunit uses a fully qualified method name as the ReSharper element ID, so a simple lookup using `Test.FullyQualifiedName` at the start of the map function can return most methods elements without having to parse anything.
* Categories should be retrieved from the `Test` using the protected `GetCategories` method, and passed into the element factory, which should update the element appropriately, and which should call the `IUnitTestElementsObserver.OnUnitTestElementChanged` method.
* The way xUnit.net encodes theory parameters in a test's `FullyQualifiedName` is completely opaque, and cannot be parsed to get more information about the theory itself. E.g. the test ID can be something like `MyNamespace.MyTestClass.MyMethod(E62628EDC980AAB)`. However, because the xUnit.net test provider doesn't discover theory tests in-editor, the name of the theory doesn't matter, and the opaque ID can be used directly as the theory element name. This also allows for a quick lookup at the start of the method.

## Limitations

The `dotnet test` protocol has a number of limitations, not least that the only way to get meaningful information is by parsing an opaque ID. Also, output is only ever reported at the end of a test run, and not while it's running. An [issue has been raised on the dotnet/cli GitHub repo](https://github.com/dotnet/cli/issues/2501).
