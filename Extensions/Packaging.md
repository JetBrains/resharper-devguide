# Packaging and Distribution

Once you’re done writing your plugin, you probably want to deploy it to end users. Starting with ReSharper 8, we provide support for clean and easy deployment of your plugins with the new Extension Manager. For these purposes, your plugin must be prepared as a [NuGet package](http://docs.nuget.org/). Also, there is a central place, the ReSharper Gallery, that can be used to host your plugins - it can be found at <http://resharper-plugins.jetbrains.com>. But you can also use you own server to deploy plugins (e.g., within your company). Repositories can also be stored in a physical folder or a shared location on the network.


Creating a package happens in the same way as you create a package for NuGet, and requires you to use the `nuget.exe` Command-Line Utility, which you can download from <http://nuget.codeplex.com/releases>. You create a `.nuspec` file and call `nuget pack` and push to the repository with:

`nuget push myextension.1.0.0.nupkg -ApiKey XXX -Source https://resharper-plugins.jetbrains.com`

> **Note** Calling `nuget pack` without specifying the `.nuspec` file causes Nuget to search for any project file (e.g., `.csproj`) or `.nuspec` file in the current directory and build the package file from the first one it finds. Of course, you can also specify the package file explicitly with `nuget pack myplugin.nuspec`.
>
> You can safely ignore the warning about the assembly being outside the lib folder. Alternatively, create your package with `nuget pack -NoPackageAnalysis`.

## Package Structure

In order to locate plugins, ReSharper searches a number of predefined locations inside the package. These locations are searched _recursively_, so you can create nested folders for keeping your plugins. Several plugins for different versions of ReSharper can be deployed in one package.

In terms of versioning, it’s possible to constrain the deployment to a specific version of ReSharper (the recommended option), or to specific versions of both ReSharper and Visual Studio (for example, if you need to take advantage of specific Visual Studio interfaces):

The general package structure should be

    {Product}\{Version}\[VsVersion]\["plugins"|"settings"|"annotations"]

Where:
* **`{Product}`** is the name of a product, e.g. "ReSharper", "dotTrace", "dotCover" or "dotPeek"
* **`{Version}`** is the version of the target product in form of "v8.0", "v8.1.1", "v8.2.3" or "vAny". This allows the package to contain mulitple files that are specific to different versions of a product. For example, a package can contain the plugin files for ReSharper 8.0 and 8.1. The special value "vAny" means the files are applicable to all versions of the product.
* **`[VsVersion]`** is an optional folder to restrict your extension to a specific version of Visual Studio. This is in the form "vsX.X", e.g. "vs10.0", "vs11.0", etc. If your plugin only works on a specific version of Visual Studio (for example, it enables ReSharper to use Visual Studio 2012's preview tab), then it should be packaged in the appropriate Visual Studio specific folder (e.g. Visual Studio S2012 is "vs11.0")
* **`["plugins"]`** is an optional subfolder where the extension manager will look to load any plugin dlls. This folder is not required if there are no plugins. Plugins **must** be in a version specific folder, e.g. `v8.1`
* **`["settings"]`** is an optional subfolder for settings to deploy in the form of a standard ReSharper .DotSettings format. These settings should not be default values for a plugin's settings - use `IHaveDefaultSettingsStream` for this. The settings can contain Live Templates or Structural Search and Replace patterns, or override other, existing settings. The package can contain only settings, and does not require a plugin. It is recommended that settings files are placed in the `vAny` folder.
* **`["annotations"]`** is an optional subfolder for external annotations that you may wish to deploy. Once again, there is no requirement for these annotations to be related to the plugin, and the package can contain only the external annotations, without any binary files. Note that annotations files must have a ".xml" file extension, and must be named after the assembly they're for, or live in a directory with that assembly's name. E.g. `annotations\myAssembly.xml` and `annotations\myAssembly\annotations.xml` are both valid. `annotations\myAssembly.ExternalAnnotations.xml` is not. Annotations should be placed in the `vAny` folder.

## Dependency on "ReSharper" package

The `.nuspec` file must declare a dependency on a package called "ReSharper". This allows an extension to declare the version requirements of the host. So, if an extension requires ReSharper 8.0, it can create a dependency on ReSharper, with a minimum version of 8.0. If it also supports 8.1, it can have a dependency of 8.0 <= ReSharper < 8.1, or `[8.0, 8.1)` in the nuspec format.

The "ReSharper" package doesn't exist as something you can download and install, but is created and managed by the Extension Manager. It refuses to install any packages that don't have this dependency, so we don't try to install NuGet packages that aren't extensions. It will also refuse to install any packages that don't satisfy the version requirements. The version used by Extension Manager is the file version of ReSharper, not the released product version. So, the 8.0 release is actually 8.0.14.856, and the 8.0.1 release is 8.0.1000.2286. These product and file versions are both displayed in the help -> about dialog. When you specify that an extension requires at least "8.0", that is actually treated as requiring at least "8.0.0.0". You can be more specific in your requirements, e.g. "8.0.1000" to target just 8.0.1.

## Pre-release packages

The Extension Manager supports NuGet's concept of pre-release packages. A NuGet package version (mostly) follows the [semver](http://semver.org) standard, which is Major.Minor.Patch, e.g. 2.0.12. If the patch includes a label, e.g. 2.0.12-beta1, then the package is considered a pre-release package, and is not shown by default. The Extension Manager and the Extension Gallery both have switches to show pre-release versions, and pre-releases are highlighted as such in the Extension Manager UI.

This mechanism allows for pre-release versions of plugins, but also allows for supporting pre-release versions of a product. It is considered good practice to mark your packages as pre-release when targetting a version of a product that is in EAP. Since ReSharper's APIs evolve and can introduce breaking changes during development, it is best that your packages are pre-release to indicate that they might not be stable with the current build.

Since a package can contain plugin files that target multiple versions of the product (e.g. ReSharper 8.0 and 8.1), you can support the EAP of 8.1 with the same package by making it pre-release. Those who want to test and use your plugin on the latest EAP version (e.g. 8.1) can opt-in to the pre-release version, and the users of 8.0 can continue using the stable version of the package, without being notified of updates. Once the product is released, you can update your package to remove the pre-release version label, and both 8.0 and 8.1 users will be prompted for update. When the 8.0 users upgrade to 8.1 of the product, they will already have the 8.1 version of the plugin installed, side by side with the 8.0 version.

Similarly, if users upgrade to a new version of a product before the plugin is updated, the package remains installed, but is not loaded (because the files are in the 8.0 folder, not the 8.1 folder). Since the package is still installed, when a new version is available on the gallery, the user is prompted to update.

## Sample Package Definition

The following is a sample package nuspec definition for deploying a plugin.

This example states that it is compatible with any ReSharper 8.1 version, any 8.2 version, but not ReSharper 8.3 (remember, this is **file** version, not product version, so it will match against e.g. 8.2.1000.4556).

It deploys 4 different versions of the plugin, that target the combinations of ReSharper 8.1 and 8.2, and are also Visual Studio 2012 and Visual Studio 2013 specific.

It also deploys external annotations that are applicable to _all_ versions of ReSharper.

```xml
<?xml version="1.0"?>
<package >
  <metadata>
    <id>MyPlugin</id>
    <version>1.0.0.0</version>
    <title>MuPlugin Title</title>
    <authors>JetBrains</authors>
    <description>MyPlugin description.</description>
    <copyright>Copyright &#x00A9; 2013 JetBrains</copyright>
    <tags>Cool Plugin</tags>
    <dependencies>
      <dependency id="ReSharper" version="[8.1, 8.3)" />
    </dependencies>
  </metadata>
  <files>
    <file src="bin\MyPlugin\VS2012\rs81\foo.vs11.dll" target="ReSharper\v8.1\vs11.0\plugins" />
    <file src="bin\MyPlugin\VS2012\rs81\foo.vs12.dll" target="ReSharper\v8.1\vs12.0\plugins" />
    <file src="bin\MyPlugin\VS2012\rs82\foo.vs11.dll" target="ReSharper\v8.2\vs11.0\plugins" />
    <file src="bin\MyPlugin\VS2012\rs82\foo.vs12.dll" target="ReSharper\v8.2\vs12.0\plugins" />
    <file src="bin\annotations\**\*.*" target="ReSharper\vAny\annotations\" />
  </files>
</package>
```

Further information about the NuSpec format can be found at [http://docs.nuget.org/docs/reference/nuspec-reference].

## Visual Studio 2003 - 2008

The NuGet based extension manager is only available for Visual Studio 2010 and later, because the NuGet.Core assembly and the user interface is only supported in Visual Studio 2010 and above. To support older Visual Studio versions, you need to make a download available that will copy the plugin dlls to the appropriate places, such as `%LOCALAPPDATA%\JetBrains\ReSharper\v8.0\plugins`. Remember that as long as the plugin is compiled as .net 3.5, and unless specific Visual Studio interfaces are used, ReSharper plugins are compatible with all versions of Visual Studio.

