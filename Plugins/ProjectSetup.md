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

## Debugging

Debugging a plugin implies spinning up an additional instance of Visual Studio with the plugin loaded. It’s important to note that you’ll be unable to debug a plugin if another version of this plugin has already been loaded. If you need to debug a plugin while _using_ that plugin for development, simply disable the original plugin in the _Options_ dialog and restart the debugging session.

Debugging a plugin is easy - there are two flags that you can pass to `devenv.exe`:

* `/ReSharper.Internal` - this flag turns on ReSharper’s internal menu, which provides many new menu items, used by the development team while building ReSharper. Several menu options are useful for plugin developers, especially the 'PSI Viewer' menu item, which displays a view of the abstract syntax tree of the current file. This flag is so useful, it's worth adding it to your default VS shortcut.
* `/ReSharper.Plugin <path to your plugin DLL>` - this tells ReSharper to load the specified DLL. You can specify the full path, but if you want to debug the plugin in your output directory, you can just specify the filename.

## Troubleshooting

If you are debugging your plugin and its breakpoints are not being triggered, first check that the plugin has actually been loaded by looking for the the list of loaded plugins in ReSharper’s _Options_ dialog. If the plugin has been loaded, look for exceptions thrown via the `/ReSharper.Internal` mode. These would typically appear in the bottom right corner of the window.

Please note that the #1 cause of plugins not debugging is lack of appropriate metadata - if previously you could mark up a context action with `ContextAction` attribute without specifying any parameters, this will no longer work, and as a result, your component will not be loaded.

The #2 cause of plugins not debugging or acting strangely is that you’ve accidentally let a ReSharper assembly be copied into the same folder as your plugin binaries. This can happen if you’ve accidentally set `CopyLocal = True` on a R# assembly reference or if, for example, you’ve included the tests-related `.Targets file` from the SDK.

## Known Issues

### Compilation Issues

*This issue concerns plugin developers who use a license file (licenses.licx) in their plugin project.* Since the license compiler (`LC.EXE`) takes as parameters the full path of each assembly reference, you may run into an exception if the sum total of all reference paths exceeds 32000 characters. Possible workarounds for this issue are:
    * Install the ReSharper SDK into a path that is as short as possible; or
    * Create your own `.Target` files, removing some of the entries that your plugin does not require.

### Debugging Issues

Some users of the SDK under Visual Studio 2010 might have problems with debugging their plugins. This is typically manifested by the fact that the plugin works and the breakpoints appear ‘healthy’ but do not fire. This issue appears due to a known Visual Studio bug that will not be fixed in VS2010. As a result, the recommended workaround for plugin developers experiencing this problem is to start up the additional instance of VS without debugging, and then using *Debug | Attach to Process…* to attach the correct CLR debugger manually. In this case, breakpoints should trigger correctly.

### Slow debugging

Using F5 to start a debug session can sometimes result in very slow debugging. It is not clear why this is - perhaps because JIT optimisations are disabled when an application is launched under a debugger. In this circumstance, you should start Visual Studio without debugging (Ctrl+F5), and attach to the process later. Performance is much improved.

