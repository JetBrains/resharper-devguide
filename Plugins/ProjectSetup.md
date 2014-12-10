# Project Setup

Plugins for the ReSharper Platform are simply .net assemblies. In order to create a plugin project, you need to:

* Create a .net class library project.
* Add a NuGet reference to the `JetBrains.ReSharper.SDK` package.
* Compile and install the project for manual testing and debugging.

## Creating the project

Simply create a .net class library assembly. The project should be a .net 4.0 assembly, which allows the plugin to run on Visual Studio 2010, which is the oldest version of Visual Studio supported by the ReSharper Platform.

> **NOTE** The only exception to this is when creating support for unit test frameworks. A unit test framework plugin includes a runner assembly that is hosted in an external process, and actually runs the tests. This needs to be a .net 3.5 assembly, so that test assemblies targeting .net 2.0 and .net 3.5 will run (obviously, if the test framework is .net 4.0 and above, the runner assembly can also be .net 4.0 and above).

## NuGet references

Your plugin project needs to add a NuGet reference to the [`JetBrains.ReSharper.SDK` package](http://www.nuget.org/packages/JetBrains.ReSharper.SDK/). This package will include all dependencies required to build plugins that extend ReSharper - both the core ReSharper Platform (e.g. language support, reference providers, etc) and the ReSharper product (e.g. navigation, unit testing support, etc.).

> **INFO** There are currently no packages available to allow extending features specific to other .net tools, such as dotCover and dotTrace. Instructions for this will be made available later. If you need instructions sooner, please raise an [issue for the documentation](https://github.com/JetBrains/resharper-devguide/issues).

The SDK package tracks the current version of ReSharper - that is, there is only one package for ReSharper plugins, and not one for each version of the SDK. EAP releases of ReSharper will have a pre-release version in the NuGet gallery, and releases will have a stable version. To target the current release of ReSharper, add a reference to the latest stable version in the gallery.

In order to target an older version of the SDK, you can add a NuGet reference to a specific version of the SDK, e.g. by using the following command in the "Package Manager Console":

```
Install-Package JetBrains.ReSharper.SDK -Version 8.2.1158
```

The specific version number can be found by looking at the package history on the gallery. The version numbers of the SDK package match the product version of ReSharper, e.g. 8.2.1158 is for ReSharper 8.2, 8.1.555 is for ReSharper 8.1, and 9.0.20141204.190166 is for ReSharper 9.0.

An existing plugin project can be updated to the latest SDK by simply updating the `JetBrains.ReSharper.SDK` package. Changes to the API are to be expected (see the guide on [Platform Versioning](../Intro/PlatformVersioning.md) for more details), and significant changes are called out in the [What's New?](../Intro/WhatsNew.md) guide.

If an existing plugin project is required to stay at a particular version, rather than be upgraded to a new version, you can specify the required version in the `packages.config` file, by setting the `allowedVersions` attribute to the required version:

```xml
<packages>
  <package id="JetBrains.ReSharper.SDK" version="8.2.1158" allowedVersions="[8.2.1158]" />
</packages>
```

## Test packages

Previous versions of the SDK included a test package - `JetBrains.ReSharper.SDK.Tests`. This package is now deprecated, and the main SDK includes the functional testing framework. See [Testing](Testing.md) for more details.

## Continuous integration

The SDK packages are intended to work with continuous integration (CI), and provide everything necessary to build your plugin. Nothing else needs to be installed on the CI server. The NuGet packages are designed to work with package restore, so the `packages` folder should not be committed into source control.

## Debugging and manually testing the plugin

Additional steps are required in order to be able to run and debug the plugin project. Please see [Running and Debugging](Debugging.md) for more details.
