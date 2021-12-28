[//]: # (title: Initial Installation)

When creating an extension, it must be installed for the first time before it can be tested or debugged. This does introduce a little bit of a chicken and egg scenario - it usually means that the extension must be packaged up and installed before any real development takes place!

The process for creating a new extension is usually:

1. Create a new .net class library project.
2. Add the `JetBrains.ReSharper.SDK` NuGet package to the project.
3. Compile.
4. Create a `.nuspec` file that describes the plugin extension, and create the `.nupkg` file from it.
5. Install the package from a local file source and debug and test.

Once the extension has initially been installed, it generally does not need to be installed again. Simply updating the files in the local ReSharper installation folder will allow ReSharper to run the new version of your plugin. The two exceptions to this are when new files are added to the extension that need to be copied to the installation folder, and also when something changes that requires re-registration with Visual Studio. Typically, this means adding new Actions - these are statically registered as menu commands with Visual Studio.

For more details on how to get a plugin installed for the first time, check the [Installing Locally](LocalInstallation.md) section, and also make sure to check the [Copy on Build](LocalInstallation_CopyOnBuild.md) guide, too.