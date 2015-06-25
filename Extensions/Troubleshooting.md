---
---

# Troubleshooting a Plugin Install

If the extension does not load, or you cannot debug your plugin, there are several things to check:

1. Ensure the [package is in the correct format](/Extensions/Packaging.md).
    1. Ensure the package has a "." in the name, or it won't install correctly.
    2. Ensure the package [includes a dependency on "Wave"](#wave-dependency).
    3. Do not include dependencies on compile time NuGet packages, e.g. `JetBrains.ReSharper.SDK`. ReSharper extension packages and NuGet library reference packages are not the same, and cannot be used interchangeably - one is used at runtime, the other at compile time.
2. Check the [installer logs](#installer-logs) to ensure the package installed correctly. The extension may have installed, and be listed in the Extension Manager as installed, but if the files are copied to an incorrect location, the extension isn't correctly installed. The easiest thing to do is search for your extension package or assembly name.
3. Ensure the plugin has a [Zone marker](/Platform/Zones/HowTo.md) defined in the project's root namespace (e.g. if the project has code in the `Foo.Bar` and `Foo.Bar.Quux` namespaces, the zone marker should live in the `Foo.Bar.ZoneMarker` class).
4. Enable [logging](/Platform/Logging.md) via the Logging options dialog page available in [Internal Mode](/Intro/InternalMode.md), and check the logs for errors, especially related to Zones.
5. Run in [Internal Mode](/Intro/InternalMode.md), and look for reported exceptions (exceptions are suppressed when not in internal mode). Even better, use a [checked build](/Intro/Tools.md).
6. Ensure the debugger has loaded the extension (look in the Debug » Windows » Modules tool window for the name of the plugin assembly). If it hasn't loaded, check the [installer logs](#installer-logs).
7. Ensure the debugger has loaded symbols for the extension (again, look in the Debug » Windows » Modules tool window for the name of the plugin, this row will include details for symbols). If the symbols are not loaded, you will not be able to set breakpoints for the plugin. 
    1. Copy the `.pdb` files for the plugin to the installation folder (`%LOCALAPPDATA%\JetBrains\Installations\{HostFullIdentifier}`).
    2. Add the symbol location to the Tools » Options » Debugging » Symbols page in Visual Studio.

> **NOTE** The `{HostFullIdentifier}` is the name of the install folder for the current host, such as `ReSharperPlatformVs12` for Visual Studio 2013. More details can be found in the [Host Identifiers](Deployment/InstallProcess/HostIdentifiers.md) section.

## Wave Dependency

The extension package needs to include a dependency on a package called "Wave". This indicates that the extension package can be installed in that version of the ReSharper Platform (shipping the collection of products that make up the Platform is known informally as a "wave"). The "Wave" package is a "virtual" package. It doesn't exist, and cannot be added to the compile time project via nuget.org. Instead, it is created dynamically by the Extension Manager, and only package extensions that depend on the correct version of the package can be installed. Furthermore, if a package doesn't depend on the correct version of "Wave", it is not shown in the list of available packages.

The dependency should be set to `[2.0]` for ReSharper 9.1, and `[1.0]` for ReSharper 9.0. It should not be set to a range (as plugins are not compatible between wave versions). The name of the dependency is case sensitive, and should be set to "Wave". A simple gotcha here is that the Extension Manager is case *insensitive*, and will display an extension package that depends on "wave" or even "WAVE". While it looks like the extension package is installed, the package is actually rejected by the installer. This is reflected in the [logs](#installer-logs).

## Installer Logs

The installer creates logs when it runs. These can be found in `%LOCALAPPDATA%\JetBrains\Shared\v02\InstallerLogXXX`, where the `XXX` is a number to allow for multiple logs (the version number in the path is the version number of the wave, currently `02` for the ReSharper 9.1 release, `v01` is for ReSharper 9.0).
