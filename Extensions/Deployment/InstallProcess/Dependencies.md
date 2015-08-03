---
---

# Dependencies

When installing the ReSharper Platform, all of the binaries are shipped in NuGet packages. First party binaries created by JetBrains are shipped in packages that reflect the logical grouping of functionality, e.g. `JetBrains.Platform.Core.Text` provides the assemblies that work with text, providing the interfaces to work with documents and text spans.

These NuGet packages are not traditional NuGet packages - like extension packages, they are not intended to be added to a project, to provide references for compilation. Instead, they are intended to ship code that is to be used at runtime. This means the layout of files in the packages does not follow the traditional `lib` folder structure. All files that are being deployed live under a `DotFiles` folder, in a flat structure - these are the files that will be copied to the installation target folder. In order to find dependencies on other assemblies, the files are in a flat folder structure - all assemblies live directly in `DotFiles`, and not in any sub folders (the packages do include sub folders, though, usually for data files. Extensions make use of this for settings and external annotations).

The ReSharper Platform also makes use of various third party assemblies, such as [Json.NET](http://www.newtonsoft.com/json), for parsing JSON. Ideally, the installer could simply use these assemblies directly from the [NuGet packages available on nuget.org](https://www.nuget.org/packages/Newtonsoft.Json/).

However, the nuget.org packages are designed to be consumed at compile time, and can contain multiple instances of the assembly, targeting different .Net Framework versions or platforms. ReSharper does not need multiple copies of the assembly - the version it was compiled and tested against is the one that must be shipped. Rather than have the installer duplicate the functionality of NuGet in choosing the correct copy to use, ReSharper creates new packages that contain the dependencies that are required. Also, the new packages have the assembly in the `DotFiles` folder, in the same format as the first party assembly packages.

Furthermore, ReSharper uses some third party assemblies that don't have NuGet packages. These are also repackaged to fit the deployment model, and make the assemblies available. They are not intended to be consumed as NuGet package references in code projects.

## Making use of third party dependencies

When creating an extension, care should be taken when adding a dependency on a third party assembly. If possible, it is best (and simpler) to avoid such dependencies - only take on a dependency when the functionality cannot be easily added to your extension's source code.

An extension should not ship a third party assembly as part of their own package.

All files in a package, first party or extension, are copied from the `DotFiles` folder in the package to the same installation folder. If more than one package tries to distribute the same third party assembly, the installer would attempt to copy all versions to the same location. The result is undefined, but could result in the wrong version (older or newer) being installed, and one or more of the extensions (or the product itself) could become unusable.

If an extension wishes to use a third party assembly, it should create a new package that contains just that assembly, living in the `DotFiles` folder, and push it to the [ReSharper Plugin Gallery](https://resharper-plugins.jetbrains.com). The name of the package should be the same as the name of the assembly, and the version of the package should reflect the version of the assembly (or the version of the assembly's nuget.org package). By using a predictable name and version, other extensions can also use the same packages.

The extension itself should add a dependency to the new third party assembly package in the extension's `.nuspec` file. E.g.

```xml
<package>
  <metadata>
    ...
    <dependencies>
      <dependency id="Wave" version="[2.0]" />
      <dependency id="Foo.Bar.ThirdParty" version="[1.3, 1.4)" />
    </dependencies>
  </metadata>
  <files>...</files>
</package>
```

If more than one extension requires the same assembly package, the installer can use NuGet's version resolution to find a common version that can be used by both extensions. It is considered best practice to add a dependency in the `.nuspec` file that includes a version range, to allow maximum flexibility.
