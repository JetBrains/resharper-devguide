---
---

# NuGet references

Your plugin project needs to add a NuGet reference to the [`JetBrains.ReSharper.SDK` package](http://www.nuget.org/packages/JetBrains.ReSharper.SDK/). This package will include all dependencies required to build plugins that extend ReSharper - both the core ReSharper Platform (e.g. language support, reference providers, etc) and the ReSharper product (e.g. navigation, unit testing support, etc.).

> **NOTE** There are currently no packages available to allow extending features specific to other .net tools, such as dotCover and dotTrace. Instructions for this will be made available later. If you need instructions sooner, please raise an [issue for the documentation](https://github.com/JetBrains/resharper-devguide/issues).

* Table of contents
{:toc}

## Versioning

The `JetBrains.ReSharper.SDK` package is used to target the current version of ReSharper. There isn't a separate package for ReSharper 8, ReSharper 9, etc. When a new version of the ReSharper Platform is released, a new version of `JetBrains.ReSharper.SDK` is released to match - the SDK package version tracks the version of the ReSharper Platform. To target the current release of ReSharper, add a reference to the latest stable version of the SDK package in the NuGet gallery. EAP versions of ReSharper will have an SDK that is released with a newer version number, and marked as pre-release.

In order to target an older version of the SDK, you can add a NuGet reference to a specific version of the SDK, e.g. by using the following command in the "Package Manager Console":

```
Install-Package JetBrains.ReSharper.SDK -Version 8.2.1158
```

The specific version number can be found by looking at the package history on the gallery. The major and minor version numbers of the SDK package match the product version of ReSharper. For example:

| Product&nbsp;version | NuGet version       |
|----------------------|---------------------|
| ReSharper 9.1        | 9.0.20150522.81235  |
| ReSharper 9.0        | 9.0.20141204.190166 |
| ReSharper 8.2        | 8.2.1158            |
| ReSharper 8.1        | 8.1.555             |

An existing plugin project can be updated to the latest SDK by simply updating the `JetBrains.ReSharper.SDK` package. Changes to the API are to be expected (see the guide on [Platform Versioning](/Extensions/PlatformVersioning.md) for more details), and significant changes are called out in the [What's New?](/Extensions/WhatsNew.md) guide.

If an existing plugin project is required to stay at a particular version, rather than be upgraded to a new version, you can specify the required version in the `packages.config` file, by setting the `allowedVersions` attribute to the required version:

```xml
<packages>
  <package id="JetBrains.ReSharper.SDK" version="8.2.1158" allowedVersions="[8.2.1158]" />
</packages>
```

## Test packages

[Test projects](/Extensions/Plugins/Testing.md) should be created as .net class library projects and should include the `JetBrains.ReSharper.SDK.Tests` package. This package includes `JetBrains.ReSharper.SDK` as a dependency. It is a small package that only includes a simple `.props` file that gets automatically included into the test project, and sets the `JetTestProject` MSBuild property to `true`. This causes the other SDK packages to alter their behaviour. Notably, it enables the "CopyLocal" flag. Normally, the SDK packages disable copy local, so the only files in the build folder are the plugin's, plus dependencies. This is because it's not possible to run the plugin without installing into a local installation of ReSharper, so it makes no sense to copy all of the ReSharper references to the build folder. Tests, on the other hand, run independently of the Visual Studio ReSharper host, and require the ReSharper references so that nunit can find the referenced assemblies.

> **NOTE** ReSharper 9.0 did not include a `JetBrains.ReSharper.SDK.Tests` package, although 8.x and 9.1 did. The ReSharper 9.0 SDK package did not modify the copy local settings, so there was no need for a test package as the files were always copied. For various reasons, this change has been rolled back to the current behaviour.

## Continuous integration

The SDK packages are intended to work with continuous integration (CI), and provide everything necessary to build your plugin. Nothing else needs to be installed on the CI server. The NuGet packages are designed to work with package restore, so the `packages` folder should not be committed into source control.

