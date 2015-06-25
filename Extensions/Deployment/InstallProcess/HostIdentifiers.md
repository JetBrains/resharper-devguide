---
---

# Host Identifiers

When the ReSharper Platform is installed, files are copied to several installation folders, one for each host. The installation folders are in the `%LOCALAPPDATA%\JetBrains\Installations\` folder, and each host is named with a Host Identifier.

The host identifier is a combination of the name and version of the host. For standalone hosts, the name is something like `dotPeek02`, while for the ReSharper Platform, the identifier comes from the host name, the Visual Studio version and optionally, an [experimental instance](/Extensions/Deployment/LocalInstallation/ExperimentlaInstance.md) name. For example, `ReSharperPlatformVs12` is the ReSharper Platform hosted in Visual Studio 2013, and `ReSharperPlatformVs14Plugins` is the ReSharper Platform hosted in Visual Studio 2015, and using the `Plugins` experimental instance.

A host identifier is used by the SDK to find the installation folder for that host. The SDK includes a [custom MSBuild task](/Extensions/Deployment/LocalInstallation/CopyOnBuild.md) that can return an installation folder when given a host identifier.

It is strongly recommended to use the custom MSBuild task rather than to try and build the installation location by hand. This is because the installation folder may not be directly named after the host identifier. When installing or updating an extension, the installer needs to update files that might be in use (because the installer has been invoked from a running instance of ReSharper). To avoid issues, the installer creates a new installation folder, based on the host identifier, but adding a `_000` or `_001` suffix to make sure it's unique. After the installation completes and the host restarts, the old installation folder is removed and the new folder is renamed to remove the suffix. However, due to file locks, this does not always happen immediately, meaning multiple folders can exist for the same host identifier. The MSBuild task handles the logic of finding out which folder is the correct one.
