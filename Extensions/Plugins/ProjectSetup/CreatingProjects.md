---
---

# Creating the project

Simply create a .net class library assembly. The project should be a .net 4.0 assembly, which allows the plugin to run on Visual Studio 2010, which is the oldest version of Visual Studio supported by the ReSharper Platform. Note that you **should not** target .net 4.5 or later, as this will mean the extension won't work in earlier versions of Visual Studio.

> **NOTE** The only exception to this is when creating support for unit test frameworks. A unit test framework plugin includes a runner assembly that is hosted in an external process, and actually runs the tests. This needs to be a .net 3.5 assembly, so that test assemblies targeting .net 2.0 and .net 3.5 will run (obviously, if the test framework is .net 4.0 and above, the runner assembly can also be .net 4.0 and above).

## Test assemblies

It is strongly recommended to create a test assembly. This should also be a .net 4.0 class library project, and should include a project reference to the main plugin assembly.

## Dependencies

The plugin can consist of multiple assemblies, however care should be taken when introducing dependencies on third party assemblies and packages. These assemblies need to be copied to the root directory of the ReSharper installation, and can easily clash and overwrite existing versions of the same files.

There are several approaches that can be taken to mitigate the risks with third party dependencies (e.g. rebuilding with a unique name, or packaging as custom NuGet packages), but it is recommended to only taken dependencies when absolutely necessary.
