---
---

# Troubleshooting a Plugin Install

If the extension does not load, or you cannot debug your plugin, there are several things to check:

1. Ensure the [package is in the correct format](../Extensions/Packaging.md).
    1. Ensure the package has a "." in the name, or it won't install correctly.
    2. Ensure the package depends on "Wave" "[2.0]" correctly. This value is case sensitive, and depends on the version of ReSharper being targeted. ReSharper 9.0 is Wave 1.0, 9.1 is Wave 2.0.
2. Ensure the [package installed correctly](../Extensions/Deployment/LocalInstallation.md#troubleshooting). The extension may have installed, and be listed in the Extension Manager as installed, but if the files are copied to an incorrect location, the extension isn't correctly installed.
3. Ensure the plugin has a [Zone marker](../Platform/Zones/HowTo.md) defined in the project's root namespace (e.g. if the project has code in the `Foo.Bar` and `Foo.Bar.Quux` namespaces, the zone marker should live in `Foo.Bar.ZoneMarker`).
4. Enable [logging](../Platform/Logging.md) via the Logging options dialog page available in [Internal Mode](../Intro/InternalMode.md), and check the logs for errors.
5. Run in [Internal Mode](../Intro/InternalMode.md), and look for reported exceptions (exceptions are suppressed when not in internal mode). Even better, use a [checked build](../Intro/Tools.md).
6. Ensure the debugger has loaded the extension (look in the Debug » Windows » Modules tool window for the name of the plugin assembly).
7. Ensure the debugger has loaded symbols for the extension (again, look in the Debug » Windows » Modules tool window for the name of the plugin, this row will include details for symbols). If the symbols are not loaded, you will not be able to set breakpoints for the plugin. 
    1. Copy the `.pdb` files for the plugin to the installation folder (`%LOCALAPPDATA%\JetBrains\Installations\{HostFullIdentifier}`).
    2. Add the symbol location to the Tools » Options » Debugging » Symbols page in Visual Studio.

> **NOTE** The `{HostFullIdentifier}` is the name of the install folder for the current host, such as `ReSharperPlatformVs12` for Visual Studio 2013. More details can be found about the `HostFullIdentifier` in the section on [Extension Deployment](../Extensions/Deployment/LocalInstallation.md#updating-the-extension-locally).
