[//]: # (title: Creating the project)

Simply create a .net class library assembly. Starting with ReSharper 2018.2, the project should be a class library that targets .NET Framework 4.6.1.

 >  Plugins for ReSharper 2016.3 til 2018.1 required .NET 4.5. Before that, ReSharper targeted .NET 4.0, in order to support Visual Studio 2010. However, .NET 4.0 is no longer supported by Microsoft, and .NET 4.5 is a recommended, in place update. This means ReSharper still supports Visual Studio 2010, but also requires .NET 4.5.
 >
 {type="note"}

## Test assemblies

It is strongly recommended to create a test assembly. This should also be a .net 4.6.1 class library project, and should include a project reference to the main plugin assembly.

## Dependencies

The plugin can consist of multiple assemblies, however care should be taken when introducing dependencies on third party assemblies and packages. These assemblies need to be copied to the root directory of the ReSharper installation, and can easily clash and overwrite existing versions of the same files.

There are several approaches that can be taken to mitigate the risks with third party dependencies (e.g. rebuilding with a unique name, or packaging as custom NuGet packages), but it is recommended to only taken dependencies when absolutely necessary.