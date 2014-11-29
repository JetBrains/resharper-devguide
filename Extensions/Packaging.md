# Packaging and Distribution

Packaging for the ReSharper Platform is very similar to the way extensions were packaged for [ReSharper 8](Packaging8.md) - it is recommended you familiarise yourself with that guide before continuing. An extension is distributed as a NuGet package, and hosted on a [NuGet gallery](https://resharper-plugins.jetbrains.com). However, because ReSharper extensions are runtime code, rather than compile time dependencies, the use case for ReSharper extension packages is very different to standard NuGet packages. Because of this, they have a different file layout internally.

Instead of adding assembly references to the `lib` folder, ReSharper extensions should add all binaries to the `DotFiles` root folder inside the package. This is different to ReSharper 8, where the top level folder was `ReSharper`. This is because extensions are now extensions to the ReSharper Platform, rather than just to ReSharper. The top level folder name has changed to reflect this.

Also unlike ReSharper 8 packages, ReSharper Platform extension packages do not provide a folder for previous versions (e.g. `ReSharper\v8.1`, `ReSharper\v8.2`, etc.). ReSharper 8.x packages tended to be updated in a timely manner for new versions of ReSharper, and maintaining previous versions in a package became unnecessary and a maintenance overhead for plugin authors, if they chose to do it at all. This means binaries should be copied to the `DotFiles` folder directly.

> **NOTE** When installing, all binary files in the package are copied to the main ReSharper Platform installation folder. This means that duplicate names should be avoided at all costs! The installer will refuse to install a package that introduces a duplicate file name.
> 
> It is recommended that plugin assembly names use a prefix to ensure their assemblies are unique, e.g. CitizenMatt.Xunit.dll.

This has an impact on dependencies - any dependent assemblies included in the extension package will be copied to the main installation directory, too, so it is easy to see how clashes can occur. Dependencies should either be ILMerge'd into the main extension, or ideally, added as NuGet package dependencies. However, the installer cannot handle dependencies which have more than one copy of the same assembly - for example, if adding `xunit.dll` using the `xunit` NuGet package, the installer does not know which copy to take, the one in `lib\net45`, `lib\portable`, etc. JetBrains currently uses private dependency packages that only contain a single copy of the assembly to work around this limitation.

The package must also add a dependency to the package called "Wave". This is the package that represents the core version of the ReSharper Platform. For ReSharper 9.0, this is Wave 1.0, so the dependency should be added as `Wave [1.0]`. Without this dependency, when the package will not be added to the correct feed on the extensions gallery, and won't be visible in the Extension Manager.

> **NOTE** Package dependencies are case sensitive, and the installer will (silently!) refuse to install a package that does not have the correct dependency on "Wave".

> **NOTE** The extension package needs to have a name that includes a "." separator, or the installer will refuse to install it. This is used by the installer to produce "Company name" metadata. The package should be in the format "Company.Package", e.g. "JetBrains.Plugins" or "CitizenMatt.Xunit".

Extensions can include settings and annotations files, by including the files in a folder in the format `Extensions\{packageId}\annotations\` or `Extensions\{packageId}\settings\`, where `{packageId}` is the full ID of the package, e.g. `Extensions\CitizenMatt.Xunit\annotations\xunit.xml` or `Extensions\CitizenMatt.Xunit\settings\template.dotSettings`. These files are copied directly, with the same folder structure into the main installation directory. This folder structure is intended to prevent name clashes. If the folder name under `Extensions` does not match the package ID directly (case insensitive), then the files will not be loaded.

> **WARNING** Annotations and settings files are not loaded by default in Beta 1. This will be fixed before RTM.
