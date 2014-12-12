# Running and Debugging an Extension

In order to manual test or debug your plugin, it must be installed into an instance of ReSharper. Currently, this is a manual step, details of which can be found in the [Local Installation](../Extensions/Deployment/LocalInstallation.md) guide (our intention is to make this easier in future versions of the SDK).

It is **strongly recommended** to install your plugin into an [experimental instance of Visual Studio](../Extensions/Deployment/LocalInstallation.html#install-to-an-experimental-instance). This allows the assembly to be updated while compiling. If the plugin being tested/debugged is installed in the main instance of Visual Studio, all instances of VS need to be closed before the plugin can be updated. If the plugin is installed in a different instance of Visual Studio to the solution used to build the plugin, the plugin's assemblies can be updated without restarting all instances of Visual Studio.

Once installed, the extension is loaded as part of ReSharper.

> **NOTE** Previous versions of ReSharper supported the `/ReSharper.Plugin` flag to dynamically load the plugin at runtime. This is no longer supported, and the plugin must be installed in order to be loaded.

## Debugging

To debug an extension, simply attach a debugger to an instance of Visual Studio that will run the plugin.

1. Set the properties of the plugin project to start Visual Studio (e.g. `C:\Program Files (x86)\Visual Studio 12.0\Common7\IDE\devenv.exe` for Visual Studio 2013). Also set the command line to run in the appropriate experimental instance, e.g. `/rootSuffix Plugins` will run Visual Studio in the "Plugins" experimental instance.
2. Start Visual Studio with `F5` or `Ctrl+F5`. Using `F5` may have performance implications as .net disables optimisations for applications started under the debugger. With an application the size of Visual Studio, this can have a notable impact on performance. In this case, use `Ctrl+F5` - Visual Studio will start quicker, you can attach (look for the `devenv.exe` process) and debug as normal.

If debugging a plugin installed into a standalone host such as dotPeek, simply change the startup executable to point to the host. The standalone applications are installed to `%LOCALAPPDATA%\JetBrains\Installations\{HostFullIdentifier}`, where `{HostFullIdentifier}` is the identifier of the host, such as `dotPeek01`, `dotTrace01`, etc. The `/rootSuffix` argument is not needed.

## Troubleshooting

If the extension does not load, or you cannot debug your plugin, there are several things to check:

1. Ensure the [package is in the correct format](../Extensions/Packaging.md).
    1. Ensure the package has a "." in the name, or it won't install correctly.
    2. Ensure the package depends on "Wave" "[1.0]" correctly. This value is case sensitive.
2. Ensure the [package installed correctly](../Extensions/Deployment/LocalInstallation.md#troubleshooting). The extension may have installed, and be listed in the Extension Manager as installed, but if the files are copied to an incorrect location, the extension isn't correctly installed.
3. Ensure the plugin has a [Zone marker](../Platform/Zones/HowTo.md) defined in the project's root namespace (e.g. if the project has code in the `Foo.Bar` and `Foo.Bar.Quux` namespaces, the zone marker should live in `Foo.Bar.ZoneMarker`).
4. Enable logging via the Logging options dialog page available in [Internal Mode](../Intro/InternalMode.md), and check the logs for errors.
5. Run in [Internal Mode](../Intro/InternalMode.md), and look for reported exceptions (exceptions are suppressed when not in internal mode).
6. Ensure the debugger has loaded the extension (look in the Debug » Windows » Modules tool window for the name of the plugin assembly).
7. Ensure the debugger has loaded symbols for the extension (again, look in the Debug » Windows » Modules tool window for the name of the plugin, this row will include details for symbols). If the symbols are not loaded, you will not be able to set breakpoints for the plugin. 
    1. Copy the `.pdb` files for the plugin to the installation folder (`%LOCALAPPDATA%\JetBrains\Installations\{HostFullIdentifier}`).
    2. Add the symbol location to the Tools » Options » Debugging » Symbols page in Visual Studio.

