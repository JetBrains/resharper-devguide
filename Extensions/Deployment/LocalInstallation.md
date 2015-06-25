---
---

# Local Installation

In order to manually test an extension, whether it's a [plugin](/Intro/CompiledExtensions.md) or a [declarative extension](/Intro/DeclarativeExtensions.md), it must first be installed into the ReSharper Platform. Once installed, existing files can generally be updated by copying a new version without re-runnning the installatione. This is a similar pattern to Visual Studio's VSIX development model.

> **NOTE** Previous versions of ReSharper allowed plugins to be run via a command line parameter (`/ReSharper.plugin`). This is no longer supported, as the ReSharper Platform now requires all extensions (including Products such as dotTrace and dotPeek) to be statically registered. Visual Studio integration plays a large role in this decision. Dynamically registering components with Visual Studio introduced issues that are not evident when statically registering (e.g. dynamically added menu items not maintaining assigned keyboard shortcuts). So, while the Component Model allows for dynamic live loading of assemblies and types, the ReSharper Platform requires static registration.

The steps to install locally are straight forward, but unfortunately, longer and more manual than we would like. We hope to improve and automate some of these steps going forward. To install locally:

1. [Install ReSharper to an experimental instance of Visual Studio](LocalInstallation/ExperimentalInstance.md) *(optional, but recommended)*.
2. [Package the extension](LocalInstallation/Packaging.md) into a custom NuGet package.
3. [Install the local copy](LocalInstallation/InstallCustomSource.md) of the extension into ReSharper, using a custom extension source.
4. Modify the project so that the plugin files are [automatically copied](LocalInstallation/CopyOnBuild.md) when the project is built.

If after following these steps, the extension doesn't appear to load, please check the [Troubleshooting](/Extensions/Troubleshooting.md) page.
