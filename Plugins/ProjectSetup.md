# Project Setup

> **Warning** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

ReSharper can be extended in several ways. It supports declarative extensions, in the form of settings files that include content like Live Templates, Structural Search and Replace patterns, or just alternative formatting and inspection defaults. It also supports a more traditional, binary based plugin, using a standard .net assembly.

## NuGet References

Your plugin project needs to add a NuGet reference to the [JetBrains.ReSharper.SDK package](http://www.nuget.org/packages/JetBrains.ReSharper.SDK/). When you use the SDK's project templates to create a project, this package is automatically added for you (even if you're not connected to the internet - the SDK includes a copy of the NuGet package). Of course, you can also add a NuGet reference manually using the NuGet reference manager inside Visual Studio.

The SDK package tracks the current version of ReSharper. In other words, the 8.2.1158 SDK package is used to target ReSharper 8.2. If you want to write plugins for a previous version of ReSharper, you can add a NuGet reference to a previous version of the SDK, e.g. ReSharper 8.1 will use the 8.1.555 SDK package. If you already have a plugin targeting 8.1, you can use NuGet to simply update your reference from 8.1.555 to 8.2.1158. During public EAP builds of new versions of ReSharper, a new NuGet package is periodically uploaded to nuget.org, but marked as pre-release. The stable version of the SDK package always reflects the latest production release of ReSharper.

Your plugin's test project should add a reference to the [JetBrains.ReSharper.SDK.Tests package](http://www.nuget.org/packages/JetBrains.ReSharper.SDK.Tests/) package. This package follows the same versioning scheme.

The assemblies included in the SDK packages are from a "checked build", providing more diagnostics while running your tests. Asserts are enabled and exceptions include more diagnostic information.

The packages are intended to work with NuGet's package restore, so you don't need to commit the packages folder into source control.

## Metadata - assembly attributes

In order for ReSharper to load the plugin, the assembly must have appropriate metadata definitions. At the very least, plugin writers are asked to define the title of the plugin, its description and authorship. This information is then displayed in the _Plugins_ tab of the _Options_ dialog.

These attributes are added to the `AssemblyInfo.cs` file by the SDK project templates:

```cs
[assembly: PluginTitle("title")]
[assembly: PluginDescription("my plugin does interesting stuff")]
[assembly: PluginVendor("my company")]
```

No further metadata is necessary as the plugin functionality is going to be picked up automatically by ReSharper via reflection.

## Known Issues

### Compilation Issues

*This issue concerns plugin developers who use a license file (licenses.licx) in their plugin project.* Since the license compiler (`LC.EXE`) takes as parameters the full path of each assembly reference, you may run into an exception if the sum total of all reference paths exceeds 32000 characters. Possible workarounds for this issue are:
    * Install the ReSharper SDK into a path that is as short as possible; or
    * Create your own `.Target` files, removing some of the entries that your plugin does not require.

