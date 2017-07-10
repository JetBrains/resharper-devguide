---
title: Running and Debugging an Extension
redirect_from:
  - /Plugins/Debugging.html
---

In order to manually test or debug your plugin, it must first be installed into an instance of ReSharper. Currently, this initial install is a manual step, details of which can be found in the [Initial Installation](/Extensions/Plugins/ProjectSetup/InitialInstallation.md) guide (our intention is to make this easier in future versions of the SDK). Once the plugin has been initially installed, it can be updated automatically by modifying the project to [copy updated files on build](/Extensions/Plugins/ProjectSetup/CopyOnBuild.md).

It is **strongly recommended** to install ReSharper and your plugin into an [experimental instance of Visual Studio](/Extensions/Deployment/LocalInstallation/ExperimentalInstance.md). This allows the assembly to be updated while compiling. If the plugin being tested/debugged is installed in the main instance of Visual Studio, all instances of VS need to be closed before the plugin can be updated. If the plugin is installed in a different instance of Visual Studio to the solution used to build the plugin, the plugin's assemblies can be updated without restarting all instances of Visual Studio.

Once installed, the extension is loaded as part of ReSharper.

> **NOTE** Previous versions of ReSharper supported the `/ReSharper.Plugin` flag to dynamically load the plugin at runtime. This is no longer supported, and the plugin must be installed in order to be loaded.

## Debugging

To debug an extension, simply attach a debugger to an instance of Visual Studio that will run the plugin.

1. Set the properties of the plugin project to start Visual Studio (e.g. `C:\Program Files (x86)\Visual Studio 12.0\Common7\IDE\devenv.exe` for Visual Studio 2013). Also set the command line to run in the appropriate experimental instance, e.g. `/rootSuffix Plugins` will run Visual Studio in the "Plugins" experimental instance.
2. Start Visual Studio with <kbd>F5</kbd> or <kbd>Ctrl</kbd>+<kbd>F5</kbd>. Using <kbd>F5</kbd> may have performance implications as .net disables optimisations for applications started under the debugger. With an application the size of Visual Studio, this can have a notable impact on performance. If this is the case, use <kbd>Ctrl</kbd>+<kbd>F5</kbd> - Visual Studio will start quicker, you can attach (look for the `devenv.exe` process) and debug as normal.

If debugging a plugin installed into a standalone host such as dotPeek, simply change the startup executable to point to the host. The standalone applications are installed to `%LOCALAPPDATA%\JetBrains\Installations\{HostFullIdentifier}`. The `/rootSuffix` argument is not needed.

> **NOTE** The [`{HostFullIdentifier}`](/Extensions/Deployment/InstallProcess/HostIdentifiers.md) is the name of the install folder for the current host, such as `ReSharperPlatformVs12` for Visual Studio 2013, or `dotPeek01`. More details can be found in the section on [Host Identifiers](/Extensions/Deployment/InstallProcess/HostIdentifiers.md).

If the plugin doesn't load, see the [Troubleshooting section](/Extensions/Troubleshooting.md).
