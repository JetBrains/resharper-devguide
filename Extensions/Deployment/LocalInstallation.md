# Local Installation

In order to manually test an extension, whether it's a [plugin](../../Intro/CompiledExtensions.md) or a [declarative extension](../../Intro/DeclarativeExtensions.md), it must first be installed into the ReSharper Platform. Once installed, existing files can be updated by copying a new version in place. This is a similar pattern to Visual Studio's VSIX development model.

> **NOTE** Previous versions of ReSharper allowed plugins to be run via a command line parameter (`/ReSharper.plugin`). This is no longer supported, as the ReSharper Platform now requires all extensions (including Products such as dotTrace and dotPeek) to be statically registered. Visual Studio integration plays a large role in this decision. Dynamically registering components with Visual Studio introduced issues that are not evident when statically registering (e.g. dynamically added menu items not maintaining assigned keyboard shortcuts). So, while the Component Model allows for dynamic live loading of assemblies and types, the ReSharper Platform requires static registration.

The steps to install locally are straight forward, but unfortunately, longer and more manual than we would like. We hope to improve and automate some of these steps going forward. To install locally:

1. Install ReSharper to an experimental instance of Visual Studio *(optional)*.
2. Package the extension into a custom NuGet package.
3. Install the local copy of the extension into ReSharper.

## Install to an experimental instance

This first step is optional, but strongly recommended for working with plugins for ReSharper. This allows for having the plugin solution open in one instance of Visual Studio, and the plugin running in another instance of Visual Studio. The instance of Visual Studio with the solution open does not keep the plugin locked in memory. This is not strictly required for declarative extensions as ReSharper will not keep a lock on .dotSettings or .xml files, however, it may be useful for testing purposes. Similarly, this is not required for plugins for standalone products, such as dotPeek.

Visual Studio's ["Experimental Instance"](http://msdn.microsoft.com/en-us/library/bb166560.aspx) feature is intended for developing and debugging Visual Studio extensions, and maintains a separate copy of the configuration needed to run Visual Studio. Each experimental instance can have an entirely different configuration, from theme and window layout to the extensions that are loaded.

>**Note** "Experimental instances" were previously known as "custom hives"

By default, the ReSharper Platform and all Product extensions (dotTrace, dotPeek, etc) are installed as a per-user extension for the main instance of Visual Studio. It is not visible in existing instances. However, it can be installed into an experimental instance to allow for a separate version of ReSharper (complete with separate versions of extensions) than what is installed in the main instance.

To install to an experimental instance, run the ReSharper unified installer, select the Options button, and enter the name of the instance. The experimental instance does not need to exist before starting the install.]

Once the install is complete, a separate instance of ReSharper has been installed to the system. The main instance of Visual Studio will continue to use the original install of ReSharper. Running Visual Studio in the new experimental instance will run with this newly installed copy of ReSharper, and only use the extensions installed into that copy.

To run Visual Studio in an experimental instance called "Plugins", run:

```
devenv.exe /rootSuffix Plugins
```

Of course, the experimental instance can be called anything, and you can install more than instance if you wish.

## Package the extension

In order to test the extension against a custom instance (or even the main instance), it needs to be installed, and needs to be packaged up in a custom `.nupkg` file. This is detailed in the [Packaging](../Deployment.md) guide.

> **NOTE** It might seem odd that a plugin needs to be packaged up and installed before it can even be run for the first time. However, the plugin must be registered with the ReSharper Platform in order to be loaded. If it's not registered, it can't be loaded. This pattern is also the same way that Visual Studio VSIX extensions work, however, the packaging is automated by custom tooling.
>
> For the first release of the ReSharper Platform, this process is manual. We hope to automate this for future releases.

## Install the local copy of the extension

Once the extension has been packaged up, and a `.nupkg` file has been created, it must be installed into the ReSharper Platform. In order to do so:

1. Start Visual Studio, ensuring you are running in the appropriate experimental instance.

    ```
    devenv.exe /rootSuffix Plugins
    ```

    Of course, if the extension is for a standalone product, start the appropriate host, e.g. dotPeek.

2. Add a custom source to the Extension Manager. Go to the Options dialog and select the Extension Manager page. Add a new source, pointing to a location on the local filesystem. Hit OK.
3. Display the Extension Manager, find and install the extension. The Extension Manager will aggregate extensions from the default gallery and the new custom source location (the local file system). The new extension will be found and displayed. After being selected for install, the ReSharper Platform will invoke the unified installer to find the extension package, resolve any dependencies and install and register the new package(s). The host (Visual Studio) will need to be restarted to pick up the changes.

> **NOTE** Again, we hope to automate this manual step for future releases.

## Updating the extension locally

Once the extension has been installed, it can be updated through the extension manager as normal, although this can be slow as the entire product is re-registered each time. To speed up testing times, updated files can be copied over the existing files. If new files are introduced, or old files renamed or deleted, the extension will need to be re-installed. Similarly, if a plugin adds new functionality that affects the integration with Visual Studio (e.g. adding new Actions), the extension will need to be re-registered (re-installed).

The ReSharper Platform installs to a location based on a host identifier. For example, the default location for installing ReSharper into Visual Studio 2013 is `%LOCALAPPDATA%\JetBrains\Installations\ReSharperPlatformVs12`. When installing dotPeek, the files are installed to `%LOCALAPPDATA%\JetBrains\Installations\dotPeek01` (where the 01 is the version number of the "wave" of the combined release of the ReSharper Platform and all products compatible with it). When installing to an experimental instance, the name of the instance is added to the host identifier: `%LOCALAPPDATA%\JetBrains\Installations\ReSharperPlatformVs12Plugins` is the location of the install to the "Plugins" experimental instance for Visual Studio 2013.

The SDK NuGet package adds a build step that can automate updating an existing install. It will copy the main assembly of a project to the install folder, given a particular host identifier, after a successful build. If no host identifier is specified, the assembly isn't copied. If an invalid host identifier is given, the custom task will list all host identifiers. The build step will also output the location of the installation folder for that host.

To add the host identifier, you need to modify the `.csproj` file (or add the identifier to the `.csproj.user` file, so it doesn't get committed to source control). You need to add a property called `HostFullIdentifier`. Make sure it's added before the import of the SDK targets. For example, to get the main assembly copied to the "Plugins" experimental instance of Visual Studio 2013, you would need to add something like:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  ...
  <PropertyGroup>
    <HostFullIdentifier>ReSharperPlatformVs12Plugins</HostFullIdentifier>
  </PropertyGroup>
  ...
</Project>
```

The build step will only copy the main assembly to the output directory. No dependencies are copied, and no items marked "Copy to output" will be copied either. If the solution consists of multiple projects that reference the SDK, each one can include the `HostFullIdentifier` to get the main assembly copied. Alternatively, dependencies and other files should be copied manually, or with another build step (the `JetBrains.CommonSdk.targets` file in the `DotFiles` folder of the `JetBrains.ReSharper.SDK` package can provide a template for implementing a custom build step to copy other files).

## Troubleshooting

The installer creates logs when it runs. These can be found in `%LOCALAPPDATA%\JetBrains\Shared\v01\InstallerLogXXX`, where the `XXX` is a number to allow for multiple logs (the version number in the path is the version number of the wave, currently 01 for the ReSharper 9.0 release).

Two important points to remember when building the package:

1. The package must depend on the `Wave` package (version 1.0 for the ReSharper 9.0 release). This is case sensitive, and the package will not be installed if the dependency does not exist. A simple gotcha is that the Extension Manager is case *insensitive*, and will display the extension even if the package is incorrectly cased, e.g. `wave`. While it looks like the extension is installed, the package is rejected. This is reflected in the logs.
2. The package name must contain a ".". The first segment of the name is used to infer a company name. Without this dot, the package is rejected. This is reflected in the logs.


