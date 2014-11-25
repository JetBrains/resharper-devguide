# Component Model and Visual Studio interfaces

> **Warning** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

The Component Model also allows access to interfaces implemented by Visual Studio. However, it is recommended that you try and avoid the use of Visual Studio interfaces, as this usually ties your plugin to a particular version of Visual Studio, and requires the plugin to run in the Visual Studio hosted version of ReSharper. Your plugin won't run in the Command Line Tools version of ReSharper. You are strongly encouraged to use ReSharper interfaces and services whenever possible.

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

ReSharper only exposes a subset of Visual Studio's interfaces in this manner. In fact, the interfaces are only exposed to support ReSharper's own integration with Visual Studio. In addition, some interfaces are wrapped, to ensure correct usage, e.g. wrapping parameters for COM interop, releasing returned COM intptr's, checking returned `HRESULT`s, etc. You should consult the whitelist below to see what interfaces are available. If it's there, you can consume it via a component's constructor.

If you need to access interfaces that aren’t exposed, you have two options:

1. Get access to the `Microsoft.VisualStudio.OLE.Interop.IServiceProvider` interface, and request it yourself
1. Set up a class to expose the required interfaces from the container

## Retrieving interfaces from IServiceProvider

First, it’s important to note that there are several `IServiceProvider` types in the CLR. We are interested in `Microsoft.VisualStudio.OLE.Interop.IServiceProvider`, which is a COM interface.

The Component Model explicitly bans this interface. If you try and consume `IServiceProvider` in your constructor, ReSharper will throw an exception, and tell you to use the `JetBrains.VsIntegration.Application.RawVsServiceProvider` wrapper class. The purpose of this class is to ensure that you know you are deliberately bypassing ReSharper and talking directly to Visual Studio. This isn't a recommended solution, but is sometimes necessary. Make sure you know what you are doing. You should always try and use either a ReSharper interface, or request the Visual Studio interface from ReSharper first - remember that ReSharper wraps some Visual Studio interfaces.

Once you have the `RawVsServiceProvider` class, you can simply use the `Value` property to get access to the interface, and query Visual Studio for a specific service. It is now your responsibility to ensure the interface is called correctly, and any COM requirements are met.

## Expose interfaces to the container

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

If you expect the interface to not be available, you can use `Optional<TInterface>` in your constructor. You can even combine the two `Lazy<Optional<T>>` which will defer resolving the underlying service until you need it, and allow it to be missing.

When using `map.QueryService`, you can enforce requiring these flags by using the `LazyOnly` and/or `Optional` methods. If a component tries to consume the service without `Lazy<T>` or `Optional<T>` respectively, the Component Model will throw an exception.

```cs
public void Register(VsServiceProviderComponentContainer.VsServiceMap map)
{
  map.QueryService<STextTemplating>.As<ITextTemplatingEngineHost>.LazyOnly().Optional();
}
```

The `Mef` and `OptionalMef` methods register the interface as `LazyOnly`, so you can *only* get it if you request `Lazy<TMefInterface>`. If there is a chance that the MEF interface will not be available, you should register it with the `OptionalMef` method, and consume it as `Lazy<Optional<T>>`.

## Default list of exposed interfaces

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

