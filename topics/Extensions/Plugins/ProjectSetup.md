[//]: # (title: Project Setup)

Plugins for the ReSharper Platform are just .net assemblies. In order to create a plugin project, you need to:

* [Create .net class libraries](CreatingProjects.md) for the plugin, and optionally (but recommended) for tests.
* Add [NuGet references](NuGetReferences.md) to the `JetBrains.ReSharper.SDK` and `JetBrains.ReSharper.SDK.Tests` packages as appropriate.
* [Install the project locally](InitialInstallation.md) in order to manually test and debug.
* Modify the project to automatically [update the installed plugin on each build](ProjectSetup_CopyOnBuild.md).

Once the project has been built and installed locally, it can be [manually run for testing or debugging](Debugging.md).