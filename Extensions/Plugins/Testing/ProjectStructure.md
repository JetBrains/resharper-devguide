---
---

# Test Project Structure

A test project is an [NUnit](http://nunit.org) based Class Library project, that targets .Net 4.0 and includes a reference to the main plugin project (ReSharper is a .net 4.0 application in order to support Visual Studio 2010 and above).

The test project should also include a reference to the `JetBrains.ReSharper.SDK.Tests` NuGet package. This package takes a dependency on the `JetBrains.ReSharper.SDK` package, as well as the `NUnit` package. The `SDK.Tests` package will ensure that all files required to run the plugin are copied to the output directory. This includes reference assemblies, but also content files, such as external annotations and CSS definition files.

> **NOTE** The normal SDK package does not copy these files, as the plugin cannot be run outside of the context of ReSharper, as installed in Visual Studio. For manual testing, the plugin should be [initially installed](/Extensions/Plugins/ProjectSetup/InitialInstallation.md) and then [copied to the installation folder on each build](/Extensions/Plugins/ProjectSetup/CopyOnBuild.md).

> **NOTE** ReSharper 9.0 did not publish a `JetBrains.ReSharper.SDK.Tests` package, primarily because the SDK contained all assemblies, including the test framework, and it would copy all files to the output folder, both for production builds and test builds. This left little else for the Tests package to do, so it was not released. In order to write tests for a 9.0 plugin, you would include `JetBrains.ReSharper.SDK` directly in the test project.

The standard project layout for a ReSharper project with tests is as follows:

```
+-- src\
|   +-- {plugin}.sln
|   +-- plugin\
|       +-- {plugin}.csproj
+-- test\
    +-- data\
        +-- nuget.config
        +-- ...
    +-- src\
        +-- {tests}.csproj
        +-- TestEnvironment.cs
        +-- ...
```

Where `{plugin}` is the name of your plugin solution and project and `{tests}` is the name of your tests project. The main plugin solution includes the plugin project and the tests project.

The `TestEnvironment.cs` file contains the bootstrap code for the test environment. This is detailed in the [Test Environment](TestEnvironment.md) section.

## Test data folder

The `test\data` folder contains sub-folders that in turn contain the data files for the functional tests defined in the tests project. These files are typically source files, as well as `.gold` files that contain the text output of the test. If the actual output of a test is different to the `.gold` file, the test fails.

## NuGet.config

The `test\data` folder also needs to contain a `nuget.config` file. When ReSharper runs tests, it creates an in-memory project, which will target a particular version of the .Net framework. To make tests repeatable on any machine, the ReSharper test framework will download NuGet packages that represent each version of the .Net framework. This prevents the need for having all versions of .Net installed in the GAC of the local machine. However, these NuGet packages are not intended to be referenced in normal projects, and so are not hosted on nuget.org, but are hosted on JetBrains infrastructure. In order to download these files for your plugin tests, you will need to create a `nuget.config` file in the root of your `data` folder. The file should look like this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <config>
    <!-- See https://docs.nuget.org/consume/nuget-config-file for config options -->
  </config>
  <packageSources>
    <!-- Clear any existing package sources -->
    <clear />

    <!-- Add the JetBrains test package gallery first.
         Then include nuget.org to download non-test packages to include in test projects -->
    <add key="jb-gallery" value="http://jb-gallery.azurewebsites.net/api/v2/curated-feeds/TestNuggets/" />
    <add key="nuget.org" value="http://www.nuget.org/api/v2/" />
  </packageSources>
  <disabledPackageSources>
    <clear />
  </disabledPackageSources>
  <packageRestore>
    <!-- Allow NuGet to download missing packages -->
    <add key="enabled" value="True" />

    <!-- Automatically check for missing packages during build in Visual Studio -->
    <add key="automatic" value="False" />
  </packageRestore>
</configuration>
```

The test packages are automatically restored the first time the tests are run, and are saved by default to the `Packages` folder in the root of the project structure (i.e. the `Packages` folder is alongside the `src` and `tests` folder in the above example - see below for details on how the project root is found, and how to customise the project structure).

The `repositoryPath` config setting can be used to override the default location of the saved test packages. This is useful if you wish to share the packages between multiple plugins (the test packages are rather large). E.g.:

```xml
<!-- ... [snip] ... -->
<config>
  <add key="repositorypath" value="C:\JetTestPackages" />
</config>
<!-- ... [snip] ... -->
```

This path can be local to the current directory (usually the location of the assembly containing the derived `ExtensionTestEnvrionmentAssembly` class), or fully qualified, and shared across multiple solutions. However, remember that using a fully qualified path may not be the best solution for an open source project, as it requires anyone forking your project to put the files in the same location.

## Customising the project structure

It is recommended that plugins use the standard project structure described above - the test framework will not require any further configuration. However, it is possible to use a different project structure, should this be necessary. The test framework will need to be told how to find the project root, and how to find the test data folder.

When trying to find the project root, the test framework will first try to use the `JetProductHomeDir` environment variable. This is not recommended, as it requires changing for each plugin you would need to build.

If the environment variable isn't set, the test framework will then look for an empty marker file called `Product.Root` in a folder above the current location of the assembly that contains the derived `ExtensionTestEnvironmentAssembly` class (e.g. `bin\Debug` of the test project). This is the method used when building the ReSharper Platform and products, which needs explicit markers to help with the build system.

If the marker file isn't found, the test framework finally looks for an assembly level attribute that specifies the relative path from the project root to the test data path (the location that contains the [`.gold` files](GoldFiles.md)). E.g. `[assembly: TestDataPathBase("test\myData")]`. If no attribute exists, the default relative path of `test\data` is used. (Note that the attribute is marked as obsolete, but is still supported. It is recommended to stick to the default project structure.)

The test framework will use the relative path to try to find the project root, by checking to see if the relative path is valid at each parent folder above the current location of the assembly that contains the derived `ExtensionTestEnvironmentAssembly` class. Once it finds the project root, it sets the `JetProductHomeDir` environment variable in the current process (the marker file method does not set the environment variable, as the project root can be easily found by looking for the marker file again).

Plugins are recommended to follow the standard project structure described above. If the project structure is different, it is recommended to use a `test\data` folder, and if that is not possible, to use the `[TestDataPathBase]` attribute.
