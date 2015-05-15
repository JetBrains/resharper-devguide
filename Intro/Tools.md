---
---

# Required Tools

You need the following tools in order to build ReSharper extensions:

* **Visual Studio** and **ReSharper**, naturally. It is strongly recommended that you install and use a "[checked build](http://resharper-support.jetbrains.com/entries/21206508-Where-can-I-download-an-old-previous-ReSharper-version-)" of ReSharper. This is identical to the production release of ReSharper except that it has extra diagnostics turned on - asserts are enabled, more context data is added to the [`Data` dictionary of thrown exceptions](http://msdn.microsoft.com/en-us/library/system.exception.data.aspx) and exceptions are reported to the user by default.
* **[ReSharper SDK](http://www.jetbrains.com/resharper/download/)**. This download adds Visual Studio project and item templates that make it easy to set up a project to build a ReSharper plugin. It also installs sample plugin projects. You need to make sure that the SDK version matches the version of ReSharper that you are targeting - e.g. use the 8.2 SDK for ReSharper 8.2.
* **ReSharper SDK nuget packages** for [assembly references](http://www.nuget.org/packages/JetBrains.ReSharper.SDK/) and [testing](http://www.nuget.org/packages/JetBrains.ReSharper.SDK.Tests/). These packages contain the assembly references for your project, as well as testing infrastructure and msbuild tools for icon creation, new language development and more. They are added automatically by the SDK project templates, or you can add them directly using the NuGet reference manager in Visual Studio.
* **[NuGet.exe](https://nuget.codeplex.com/)**. ReSharper extensions are NuGet packages, so you'll need `nuget.exe` in order to create the packages from a `.nuspec` file.

## Note on .Net Framework Version

ReSharper targets Visual Studio 2005, 2008, 2010, 2012 and 2013. As such, it is a .net 3.5 application, although parts are .net 4. In order to support ReSharper in all Visual Studio versions, it is recommended that plugins are also .net 3.5 assemblies. However, ReSharper's NuGet based extensions only work on .net 4 (since NuGet itself is .net 4), so if you are only distributing your extension as a NuGet based extension, then you can safely target .net 4. Note that ReSharper in Visual Studio 2005 and 2008 still supports plugins, but they have to be installed manually.

