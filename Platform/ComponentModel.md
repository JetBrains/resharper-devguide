# Component Model

ReSharper has a very composable architecture, which allows for a loosely coupled, easily extensible design. Functionality is implemented in terms of components that advertise themselves to the Component Model, which in turn is responsible for lifetime management, as well as wiring up the inter-dependencies of the components. The Component Model will look for classes marked with specific attributes, which declare the lifetime scope of the component. Dependencies are declared as constructor arguments. At the appropriate time, the Component Model will create new instances of the components, ensuring all dependencies are created first, and passed into the constructor.

> **Note** If you are familiar with the concept of [Inversion of Control](http://en.wikipedia.org/wiki/Inversion_of_control) and [Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection), ReSharper's Component Model follows these patterns, and implements an IoC Container to automatically create and wire up dependendencies

This loosely coupled design allows for easily extending ReSharper - new components can easily be advertised to the Component Model, and services are available for consumption by declaring constructor arguments. It is not possible to integrate with ReSharper without using and understanding the Component Model. 

Conceptually, the Component Model is very similar to Microsoft's [Managed Extensibility Framework (MEF)](http://msdn.microsoft.com/en-us/library/dd460648.aspx) although there are some fundamental differences, such as the ability to live load and unload new components, during runtime.

The two most interesting component types that ReSharper supports are the `ShellComponent` and the `SolutionComponent`.

## Shell Components

A *Shell Component* is a class that gets created when ReSharper starts. Shell components are typically marked with the `[ShellComponent]` attribute. It has essentially the same lifetime as ReSharper itself. Shell components are effectively singletons. They are most useful when creating a service that is not solution specific. For example, Live Template macro definitions are shell components. They do not use or depend on anything related to a solution, and do not change for the lifetime of the shell.

To define a shell component, simply write:

```cs
[ShellComponent]
public class MyClass
{
  // ...
}
```

ReSharper will create an instance of this class when the shell starts up.

If a shell component needs to talk to other components, it can simply add them as a constructor argument, and the Component Model will ensure that the dependent component is created first, and passed in on construction. The dependent component must have the same lifetime scope or longer, so a shell component can only request another shell component.

```cs
[ShellComponent]
public class SomeOtherClass
{
  private MyClass myClass;

  public SomeOtherClass(MyClass myClass)
  {
    this.myClass = myClass;
  }
}
```

Constructor injection is the preferred means of accessing dependencies. However, there are some cases where this isn't possible - for example, Action handlers aren't created by the Component Model. In this case, you can use the Service Locator pattern and ask for the dependency:

```cs
public void SomeMethod()
{
  // Shell.Instance is a static property
  var myClass = Shell.Instance.GetComponent<MyClass>();
}
```

Note, however, that `Shell.Instance.GetComponent` will only return shell components, not solution components.

## Solution Components

*Solution components* are components that are tied to the lifetime of a solution. They are created each time a new solution is opened, and use the `[SolutionComponent]` attribute. Again, constructor injection is the preferred means of satisfying dependencies. A `SolutionComponent` can depend on both `SolutionComponent` and `ShellComponent` instances, because shell components have a longer lifetime than solution components.

You can also use the Service Locator pattern to ask for solution components. There are several ways of getting at the solution level Component Model, usually with a `GetComponent` or `GetComponents` extension method. For example, the `IDataContext` object passed to action handlers can be used to retrieve components, or the `ISolution` and `IProject` interfaces have extension methods. Again, when asking for a component directly, you can ask for a solution component or shell component. For example:

```cs
var solution = context.GetData(JetBrains.ProjectModel.DataContext.DataConstants.SOLUTION);
var dtm = solution.GetComponent<DocumentTransactionManager>();
```

One very important object that can be injected or asked for is `ISolution`. This allows a solution level component to get at its current context - the solution it's been created for.

## Other types of component

ReSharper supports several types of component, not just shell and solution components. For completeness, it also supports `EnvironmentComponent`, `ShellInstanceComponent` and `ProjectComponent`.

A `ShellComponent` has the same lifetime as ReSharper, but this might not be the same lifetime as Visual Studio - if someone suspends and resumes ReSharper, all shell components are released and then recreated. An environment component is created when ReSharper first starts up and lasts for the duration of the hosting application (such as Visual Studio), regardless of suspend and resume. It is intended for ReSharper's integration infrastructure, and generally speaking, should not be used by plugin authors.

`ShellInstanceComponent` is also an infrastructure component, created as the solution is opened. Again, there should be little need for it to be used by plugins, use `ShellComponent` instead.

`ProjectComponent` allows for a class to be created for each project that's loaded. This means, unlike shell and solution components, it's not a singleton - there will be one instance for each loaded project. If a project is unloaded or removed, the component is released. If a new project is added to the current solution, a new component is created. The Component Model can inject an instance of `IProject` to give the component some context. It's generally used for project level settings and properties (e.g. project level Live Templates, or C# language level). It is rarely used other than this.

There are other component attributes, but they tend to derive from `ShellComponent` or `SolutionComponent`, e.g. `PsiComponent` or `CodeCleanupModuleAttribute`. These are treated the same as normal `ShellComponent` and `SolutionComponent` instances, but the derived attributes allow for grouping and passing more information to consumers. For example, a common pattern is to annotate a derived class with the `MeansImplicitUse` and `BaseTypeRequired` attributes, to show that any class marked with this attribute is used, and must implement a specific interface.

## Component cleanup

A component is tied to a particular lifetime - the lifetime of the shell for `ShellComponent` instances and the lifetime of the solution for `SolutionComponent` instances. When the solution is closed, or the shell terminates, the component is now out of scope, and is available for garbage collection. If you need to do any explicit cleanup that can't be handled by garbage collection, you can implement `IDisposable` and the Component Model will call `Dispose` for you. 

Alternatively, you should inject an instance of `Lifetime` into your constructor, and register callbacks with it. The `Lifetime` object is terminated at the end of the lifetime scope of the component, and will call any registered callbacks at that time. `Lifetime` is a very powerful construct for lifetime management, and is [documented separately](Lifetime.md).

You should ensure that you are not holding any references that are outside the scope of your component. For example, a shell component should not acquire and cache an instance of `ISolution`, or any other solution component, as this will cause memory leaks.

## Dependency failures

One downside to wiring up an application through the Component Model is that errors can cascade. For example, a shell component with a constructor that takes a solution component cannot be created, and the Component Model will throw an exception. But a second shell component that depends on this failed first shell component will now fail to be created, and so on, potentially destabilising the entire application. Care should be taken to require dependencies appropriately, and not cause exceptions in constructors.

## Lazy and optional acquisition

Dependencies can be declared as lazy or optional.

A lazy dependency will only be resolved at the time of use, rather than at the time of creation of the owning component. Simply use `JetBrains.Util.Lazy<ComponentType>` instead of `ComponentType` when querying or injecting a component. Note that this does not work with the BCL's `System.Lazy` type, as this is a .net 4 type, and ReSharper is a .net 3.5 application (although it runs in .net 4 when hosted in VS2010 and VS2012).

An optional dependency is resolved at component creation time (same as normal components), but can be missing, and if so, will be null. You declare an optional dependency by requesting `Optional<ComponentType>` instead of `ComponentType`.

## Acquiring Visual Studio interfaces

ReSharper also allows access to interfaces implemented by Visual Studio. However, it is recommended that you try and avoid the use of Visual Studio interfaces, as this usually ties your plugin to a particular version of Visual Studio, and requires the plugin to run in the Visual Studio hosted version of ReSharper. Your plugin won't run in the Command Line Tools version of ReSharper. You are strongly encouraged to use ReSharper interfaces and services whenever possible.

If you still need to get an interface implemented by Visual Studio, or by a Visual Studio extension, you can do this in several ways. The easiest is to simply inject the interface as a constructor parameter to your plugin component, or retrieve it from the Shell’s container. For example, to get to VS's Output Window interface `IVsOutputWindow`, you can acquire it with the following:

```cs
Shell.Instance.GetComponent<Lazy<IVsOutputWindow>>()
```

Note the use of `Lazy<T>`. Many of the Visual Studio interfaces can only be resolved using `Lazy<T>`, to allow for delay loading VS packages and assemblies, or `Optional<T>`, because there is no guarantee that interface is available - perhaps the required VS package isn't installed.

It is recommended that you use `Lazy` and `Optional`. If the plugin is run outside of the Visual Studio environment (e.g. in the Command Line Tools), the Visual Studio dependencies will not be available. It is better for your component to handle this gracefully than for the Component Model to be unable to create your component. An alternative is to mark your component as only being able to run in the Visual Studio environment, by passing in the `ProgramConfigurations.VS_ADDIN` flag to the component attribute. However, this can have a "viral" effect. If this component isn't created because you're not running inside Visual Studio, any other component that depends on this must also include the `VS_ADDIN` flag, or the Component Model will be unable to create it, also. A better solution is the use of `Optional`, `Lazy`, or perhaps moving the component into a Visual Studio specific assembly that is only loaded in the Visual Studio environment.

```cs
[ShellComponent(ProgramConfigurations.VS_ADDIN)]
public class MyShellComponent { ... }

[SolutionComponent(ProgramConfigurations.VS_ADDIN)]
public class MySolutionComponent { ... }
```

ReSharper only exposes a subset of Visual Studio's interfaces in this manner. In fact, the interfaces are only exposed to support ReSharper's own integration with Visual Studio. In addition, some interfaces are wrapped, to ensure correct usage, e.g. wrapping parameters for COM interop, releasing returned COM intptr's, checking returned HRESULTs, etc. You should consult the whitelist below to see what interfaces are available. If it's there, you can consume it via a component's construcutor.

If you need to access interfaces that aren’t exposed, you have two options:

1. Get access to the `Microsoft.VisualStudio.OLE.Interop.IServiceProvider` interface, and request it yourself
1. Set up a class to expose the required interfaces from the container

### Retrieving interfaces from IServiceProvider

First, it’s important to note that there are several IServiceProvider types in the CLR. We are interested in `Microsoft.VisualStudio.OLE.Interop.IServiceProvider`, which is a COM interface.

The Component Model explicitly bans this interface. If you try and consume `IServiceProvider` in your constructor, ReSharper will throw an exception, and tell you to use the `JetBrains.VsIntegration.Application.RawVsServiceProvider` wrapper class. The purpose of this class is to ensure that you know you are deliberately bypassing ReSharper and talking directly to Visual Studio. This isn't a recommended solution, but is sometimes necessary. Make sure you know what you are doing. You should always try and use either a ReSharper interface, or request the Visual Studio interface from ReSharper first - remember that ReSharper wraps some Visual Studio interfaces.

Once you have the `RawVsServiceProvider` class, you can simply use the `Value` property to get access to the interface, and query Visual Studio for a specific service. It is now your responsibility to ensure the interface is called correctly, and any COM requirements are met.

### Expose interfaces to the container

You can implement a class that will be called by ReSharper during initialisation that can register additional Visual Studio interfaces. It needs to be marked with the `WrapVsInterfaces` attribute, and implement the `IExposeVsServices` interface:

```cs
[WrapVsInterfaces]
public class ExposeMyServices : IExposeVsServices
{
  public void Register(VsServiceProviderComponentContainer.VsServiceMap map)
  {
    // For example:
    // map.QueryService<SComponentModel>.As<IComponentModel>();
    // or map.Mef<IOutliningManagerService>();
    // or map.OptionalMef<IOutliningManagerService>();
    // or map.Wrapper(interface => GetInterestingInterface(map, interface));
  }
}
```

This class allows you to expose one or more interfaces that comes from a service. The first example gets the `IComponentModel` interface from the MEF `SComponentModel` service, allowing you to call `IComponentModel.GetService` to retrieve a service exposed by MEF. The next two examples show an easier way of doing this, by adding support for a particular MEF exposed interface. The `OptionalMef` call is the same as the `Mef` call, but requires the interface to be wrapped in `Optional<T>`, or it won't be found. Finally, the Wrapper method calls a custom function to resolve the interface. Use of this method is discouraged.

One important thing to note is that you can only register an interface once. If there are two plugins both registering the same interface, only the first registration (arbitrarily chosen) will be used. The second is ignored. So it is important not to do anything “fancy” in a custom function - it might never get called! Also, while the second registration is ignored and the registration method completes successfully, ReSharper handles and logs an exception. Since these exceptions are displayed to the user in the exception reporter, it is a good practice to check if the interface is already registered before trying to register yourself. You can use the following extension method to do this (and doesn't interfere with lazy instantiation):

```cs
public static bool IsRegistered<T>(this VsServiceProviderComponentContainer.VsServiceMap map)
{
  return map.Resolve(typeof (T)) != null;
}
```

These interfaces are now exposed to ReSharper’s container, and can be injected into the constructor, or retrieved via the Shell’s `GetComponent` method.

You should use `Lazy<TInterface>` when consuming a Visual Studio interface, so that Visual Studio packages are not loaded until they are used. For example, accepting a parameter of `IEditorOperationsFactoryService` will cause the VS editor to be loaded into memory as soon as your plugin is loaded, which is at application startup, rather than when the editor is first needed, which might be after a solution has loaded.

If you expect the interface to not be available, you can use `Optional<TInterface>` in your constructor. You can even combine the two `Lazy<Optional<T>>` which allows you to defer checking for a value until you need it, and then allow it to be missing.

The interface is being registered as `LazyOnly`, which means you can *only* get it if you request `Lazy<TMefInterface`. If there is a chance that the MEF interface is not available, you should request `IComponentModel` (which isn't exposed by default, see example above) and calling `IComponentModel.GetService<>` yourself, wrapped in a try/catch block.

### Default list of exposed interfaces

For ReSharper 8, the default whitelist of supported interfaces is as follows:

* `IVsSolutionBuildManager`, `IVsSolutionBuildManager2`, `IVsSolutionBuildManager3`
* `IVsTrackProjectDocuments2`
* `IVsTextManager`, `IVsTextManager2`, `IVsHiddenTextManager`
* `ILocalRegistry`, `ILocalRegistry2`
* `IOleComponentManager`
* `IVsUIShell`, `IVsUIShell2`
* `Help`, `Help2`
* `IVsProfferCommands`, `IVsProfferCommands2`, `IVsProfferCommands3`
* `IVsFontAndColorCacheManager`
* `IVsQueryEditQuerySave2`
* `IVsCodeDefView`
* `IVsDebugger`, `IVsDebugger2` (Lazy only)
* `DTE`, `DTE2`
* `IVsCmdNameMapping`
* `IProfferService`
* `IUIHostLocale`
* `IVsAppCommandLine`
* `IVsClassView`
* `IVsExternalFilesManager`
* `IVsFileChangeEx`
* `IVsFontAndColorStorage`
* `IVsLaunchPad`, `IVsLaunchPad2`
* `IVsLinkedUndoTransactionManager`
* `IVsObjBrowser` (Lazy only)
* `IVsObjectManager2` (Lazy only)
* `IVsOutputWindow` (Lazy only)
* `IVsRegisterEditors`
* `IVsRegisterPriorityCommandTarget`
* `IVsSolution`
* `IVsUIShellOpenDocument`
* `IVsWebBrowsingService`
* `IVsActivityLog`
* `IVsExtensibility3`
* `IVsWebProxy`

Also, the following interfaces are exposed when running in Visual Studio 2010:

* `IVsEditorAdaptersFactoryService`
* `IOutliningManagerService`
* `IEditorOperationsFactoryService`
* `IClassificationTypeRegistryService`
* `IClassificationFormatMapService`
* `IViewTagAggregatorFactoryService`
* `IIncrementalSearchFactroyService`
* `IVsSolution4`
* `IVsUIShell4`
* `IVsThreadedWaitDialogFactory`
* `IVsBuildManagerAccessor`
* `IVsFindManagerHelper`
* `IVsPackageInstallerServices` (Lazy/Optional only)
* `IVsPackageInstallerEvents` (Lazy/Optional only)
* `IVsPackageInstaller` (Lazy/Optional only)
* `IVsPackageUninstaller` (Lazy/Optional only)

And Visual Studio 2012:

* `IVsTaskSchedulerService`
* `IVsUIShell5`
* `IVsFileChangeExPrivate`
* `IVsAppContainerProjectDeploy`
* `IVsDebugger4`

And Visual Studio 2013:
* `IPeekBroker` (Lazy/Optional only)

Furthermore, the following classes are used instead of accessing the VS interfaces directly:

* `RawVsServiceProvider` (`IServiceProvider` - use advisedly!)
* `JetBrains.VsIntegration.Interop.Shim.Shell.IVsMonitorSelection`
* `Interop.Shim.Shell.IVsRunningDocumentTable`
* `Interop.Shim.Shell.IVsShell`
* `JetBrains.VsIntegration.Application.VsUIHostCommandDispatcher` (`SUIHostCommandDispatcher`’s `IOleCommandTarget`)

