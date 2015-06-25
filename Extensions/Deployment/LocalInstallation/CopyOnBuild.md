---
---

# Copy Plugin on Build

When developing a plugin, it must [first be installed](/Plugins/ProjectSetup/InitialInstallation.md) in order to manually test and debug it. While it is possible to repackage and update through the extension manager on each compile, this is an overhead that is not necessary for day-to-day development. Instead, the files in the installation folder can be overwritten, and ReSharper will use the new version when the host (e.g. Visual Studio) is restarted.

This method works as long as there are no changes that require re-registering the extension with ReSharper or Visual Studio. In this case, the extension should be repackaged and updated through the Extension Manager, using a custom source to install from a local folder.

* An extension requires re-registering with ReSharper if files are added, removed or renamed, as ReSharper needs to know what files to load.
* An extension requires re-registering with Visual Studio if any integration points with Visual Studio are modified. Typically, this means if any Actions are created, modified or removed, or the menu items they correspond to are moved or updated, as these are translated at install time to Visual Studio commands, and statically registered.

## Automating the copy

The SDK NuGet package adds a build step called `CopyPlugin` that can automate updating an existing install when a project is successfully built. This target will execute after the `AfterBuild` target, and will copy the main assembly of the project to the install folder of a specified host.

The MSBuild target uses a custom task to map between a [Host Identifier](/Extensions/Deployment/InstallProcess/HostIdentifiers.md) and an installation folder. The `HostFullIdentifier` MSBuild property needs to be set in order for the target to know where to install the project's assembly. It is not set by default as the SDK does not know which host instance you wish to install into.

The value of `HostFullIdentifier` is the host name and version, such as `dotPeek02` (where 02 is the version of the wave for this instance of dotPeek) or `ReSharperPlatformVs12`, which is the host identifier for the ReSharper Platform, as hosted in Visual Studio 2013. If you are using [experimental instances](/Extensions/Deployment/LocalInstallation/ExperimentalInstance.md), the instance name is added to the end of the host identifier. So the `Plugins` experimental instance for Visual Studio 2013 would have a host identifier of `ReSharperPlatformVs12Plugins`.

The simplest way to find out what host identifiers are available on your system is to set the `HostFullIdentifier` property to an invalid value and run the build. The MSBuild task will print out a list of available identifiers, and you can update to the correct value. See the [Host Identifiers](/Extensions/Deployment/InstallProcess/HostIdentifiers.md) section for more details.

To specify the host identifier property, you need to modify the `.csproj` file. If you wish to use the same host identifier as other members of your team, you can put this directly in the `.csproj` file and commit to source control. If your team wants to deploy to different hosts (perhaps you are using different [experimental instances](/Extensions/Deployment/LocalInstallation/ExperimentalInstance.md)), it might be better to place your MSBuild property in the `.csproj.user` file, which is included into the `.csproj` file, but not committed to source control.

> **NOTE** If editing the `.csproj` file, make sure the property is set up before the statements that import the SDK targets files.

For example:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  ...
  <PropertyGroup>
    <HostFullIdentifier>ReSharperPlatformVs12Plugins</HostFullIdentifier>
  </PropertyGroup>
  ...
</Project>
```

## Copying multiple files

The build step will **only copy the main assembly** to the output directory. No dependencies are copied, and no items marked "Copy to output" will be copied either. This is because the SDK does not know what should be copied to the installation folder - this is really the concern of the `.nuspec` file. There are also [complications with dependencies](/Extensions/Deployment/InstallProcess/Dependencies.md) that need to be considered.

If there are multiple projects that need to be copied, each project can set up the `HostFullIdentifier` property (or include a common `.targets` file that does it for all projects) to get the main assembly of that project copied.

Alternatively, the project can make use of the MSBuild tasks to provide its own custom copy task, and provide its own logic for selecting what files should be copied to where. E.g.

```xml
<Target Name="CopyAllPluginFiles" AfterTargets="AfterBuild" Condition="'$(HostFullIdentifier)' != ''">
  <InstalledProductsDiscoveryTask HostFullIdentifier="$(HostFullIdentifier)">
    <Output TaskParameter="Directories" ItemName="InstallFolder" />
  </InstalledProductsDiscoveryTask>

  <!-- InstallFolder is now the location of the host -->
  <Message Text="InstallFolder: @(InstallFolder)" Importance="High" />

  <!-- Copy any files in the AllFilesToCopy property to the host installation folder.
       The AllFilesToCopy property needs to be populated, based on your own criteria -->
  <Copy SourceFiles="@(AllFilesToCopy)" DestinationFolder="@(InstallFolder)" Condition="Exists('@(InstallFolder)')" />
</Target>
```

All files should be copied to the installation folder in the same layout that they are saved in the `DotFiles` folder of the NuGet package. That is, assemblies live in the `DotFiles` folder, so should be copied to the root of the installation folder. Settings and external annotations are saved to `DotFiles\Extensions\{packageId}\settings\` and `DotFiles\Extensions\{packageId}\annotations\`, and should be copied to the `Extensions\{packageId}\settings\` and `Extensions\{packageId}\annotations\` folders in the root of the installation folder.
