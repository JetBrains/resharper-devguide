---
---

# Required Tools

You need the following tools in order to build ReSharper extensions:

* **Visual Studio** and **ReSharper**, naturally.
* **[NuGet.exe](https://nuget.codeplex.com/)**. ReSharper extensions are NuGet packages, so you'll need `nuget.exe` in order to create the packages from a `.nuspec` file.

If you wish to build a compiled extension (plugin), you will also need:

* A **"checked build"** of ReSharper. This is a build of ReSharper that is identical to the production version, except it is compiled with extra diagnostics enabled. The code contains asserts that verify assumptions during development of the product, and are not needed during day-to-day usage, but are very useful when developing extensions. Also, a checked build also includes more contextual data when an exception is thrown, by storing information in the [`Exception.Data` dictionary](http://msdn.microsoft.com/en-us/library/system.exception.data.aspx). A checked build can be [downloaded from here](http://resharper-support.jetbrains.com/entries/21206508-Where-can-I-download-an-old-previous-ReSharper-version-).
* **ReSharper SDK nuget packages** for [assembly references](http://www.nuget.org/packages/JetBrains.ReSharper.SDK/) and [testing](http://www.nuget.org/packages/JetBrains.ReSharper.SDK.Tests/). These packages contain the assembly references for your project, as well as testing infrastructure and msbuild tasks for icon creation, new language development and more.

