# Debugging

> **Warning** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

Debugging a plugin implies spinning up an additional instance of Visual Studio with the plugin loaded. It’s important to note that you’ll be unable to debug a plugin if another version of this plugin has already been loaded. If you need to debug a plugin while _using_ that plugin for development, simply disable the original plugin in the _Options_ dialog and restart the debugging session.

Debugging a plugin is easy - there are two flags that you can pass to `devenv.exe`:

* `/ReSharper.Internal` - this flag turns on ReSharper’s internal menu, which provides many new menu items, used by the development team while building ReSharper. Several menu options are useful for plugin developers, especially the 'PSI Viewer' menu item, which displays a view of the abstract syntax tree of the current file. This flag is so useful, it's worth adding it to your default VS shortcut.
* `/ReSharper.Plugin <path to your plugin DLL>` - this tells ReSharper to load the specified DLL. You can specify the full path, but if you want to debug the plugin in your output directory, you can just specify the filename.

### Troubleshooting

If you are debugging, and your plugin's breakpoints are not being triggered:

1. First check that the plugin has been loaded, by looking in the list of loaded plugins in ReSharper's Options dialog.
2. Check for exceptions having been thrown while loading the plugin - check for the exclamation mark icon in the bottom right of the status bar. Note that you need to either have a [checked build](../Intro/Tools.md), or be in [Internal Mode](../Intro/InternalMode.md) for exceptions to be reported.
3. Make sure that there are no ReSharper assemblies copied to the output folder. When loading a plugin from a folder, the Component Model will try to load all assemblies in a folder, and it will fail if it tries to register a second set of ReSharper components (any component that was expecting a single instance will suddenly have two instances, and the Component Model will be unable to satisfy requests for components). Make sure the "copy local" settings are set to `false` for ReSharper assemblies, and `true` for the plugin.

### F5 debugging not working

If the plugin is a .net 3.5 assembly, and the host application is .net 4 (e.g. VS2010 or above), then breakpoints won't work when the application is started with F5, even though they appear "healthy". The workaround is to use "Attach to process" to start debugging, rather than letting Visual Studio start the debug session with F5.

### Slow debugging

Using F5 to start a debug session can sometimes result in very slow debugging. It is not clear why this is - perhaps because JIT optimisations are disabled when an application is launched under a debugger. In this circumstance, you should start Visual Studio without debugging (Ctrl+F5), and attach to the process later. Performance is much improved.


