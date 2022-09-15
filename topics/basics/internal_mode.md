[//]: # (title: Internal Mode)

ReSharper supports an "internal mode" that enables a number of extra features that are normally hidden from end users. These are diagnostic and testing features used by the dev team, and most users will never need to use, however, they can be very useful for extension authors.

Another useful feature is that internal mode also enables exception reporting. In production builds, exceptions are silently logged and not reported to the end user. Checked builds (such as EAP builds) have exception reporting enabled by default, and also include extra contextual information in the [exception's `Data` dictionary](http://msdn.microsoft.com/en-us/library/system.exception.data(v=vs.110).aspx). [Checked builds](Tools.md) are highly recommended when developing extensions.

## Enabling internal mode

Internal mode can be enabled using the "Internal" command line switch. When used with standalone tools such as dotPeek, this means starting the application with `/Internal` in the command line. Since ReSharper is a hosted application, the command line switch needs a prefix to prevent clashes with other Visual Studio command line switches - `/ReSharper.Internal`. For example, to start dotPeek with internal mode enabled:

```text
dotPeek.exe /Internal
```

And for ReSharper:

```text
devenv.exe /ReSharper.Internal
```

Note that these values can be added to the command line of a shortcut on the desktop or pinned to the taskbar - right click the file, show properties and edit the command line in the "Shortcut" tab. When pinned to the taskbar, right click the icon and then right click on the name of the application and select "Properties", e.g. "Visual Studio 2013"

## Available features

Once enabled, ReSharper adds the "Internal" menu to the main "ReSharper" menu. This menu provides lots of items, some of which are only useful to the development team. Other items are more useful for extension authors. Some highlights:

* **PSI Browser** displays a window that shows the current file's PSI abstract syntax tree in a tree view. Very useful for seeing the structure of a file as an AST.
* **PSI Viewer** is similar to the PSI browser, but can create PSI trees based on arbitrary text in a text box. This is very useful for seeing what tree a particular code construct will create.
* **PSI Module Browser** is a tree view of all of the PSI modules in the currently loaded solution. A module is a container for anything that can be referenced, usually source files or referenced assemblies. Each project is a module, as is each referenced assembly. The module browser lists all of the modules, and shows who is referencing them. It also shows the internal modules that ReSharper sets up to add references that are implicit in the project (for example, modules that contain definitions for default global JavaScript objects).
* **Code behind** displays a "code behind" view of any secondary PSI trees of the current file. For example, HTML files maintain a hidden, generated, secondary file to represent all of the JavaScript sections in the file. ReSharper can map between the main file and the secondary file, supporting "islands" of one language inside another language. The secondary file can give more context to the islands in the primary file to aid code completion, navigation, etc.
* **Settings Store View** allows examining the value of settings, what settings files are currently mounted, and can view the settings schema.
* **Show Themed Icons** provides a tool window to browse and search the icons registered with the [`ThemeIconManager`](Theming_Icons.md), which can be useful to find icons for use in extensions.
* **Show Change Manager Graph in yEd** will create a graph file to be opened in the [yEd graphing application](http://www.yworks.com/en/products/yfiles/yed/) showing change providers, and how they are registered.

Other items are available, these are just some of the more interesting ones for extension developers.

## Options

As well as adding new menu items, enabling internal mode will add new options to the Options dialog. Again, some items are only interesting to the development team, but some are useful for extension developers. For example:

* **Tools → Unit Testing** gets two new options, **Enable Debug** and **Enable Logging**. The "Enable Debug" option will cause the external test runner process to display a message box before running any tests, giving the developer time to attach a debugger.
* The **Internal** page provides lots of interesting options, most of which are only interesting to the development team (the "Enable Zones" check box falls into this category - it analyses your source code for correct [Zone](Platform_Zones.md) usage, however it doesn't analyse referenced code, so it's not useful for extension developers. This functionality will be added by a separate extension in the future).
* The **Logging** page allows for creating and editing logging configuration files. The generated configuration file provides comments on how the file works.