---
---

# Dependencies

When installing the ReSharper Platform, all of the binaries are shipped in NuGet packages. First party binaries created by JetBrains are shipped in packages that reflect the logical grouping of functionality, e.g. `JetBrains.Platform.Core.Text` provides the assemblies that work with text, providing the interfaces to work with documents and text spans.

These NuGet packages are not traditional NuGet packages - like extension packages, they are not intended to be added to a project, to provide references for compilation. Instead, they are intended to ship code that is to be used at runtime. This means the layout of files in the packages does not follow the traditional `lib` folder structure. All files that are being deployed live under a `DotFiles` folder, in a flat structure - these are the files that will be copied to the installation target folder. In order to load dependent assemblies, the files are in a flat folder structure - all assemblies live directly in `DotFiles`, and not in any sub folders (the packages do include sub folders, though, usually for data files. Extensions make use of this for settings and external annotations).

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

## Transitive dependencies

Extensions needs to specify all of their transitive dependencies in their own `.nuspec` file. That is, if an extension required package `A`, and package `A` in turn requires package `B`, then the extension should list both package `A` and `B` as dependencies in its `.nuspec` file.

This is due to a technical limitation in the NuGet gallery software used to distribute ReSharper extensions. All extension package are uploaded to the same gallery, regardless of which version of ReSharper they target. When ReSharper requests the list of known extensions from the gallery, it does so on a "curated feed". This curated feed is an automatically maintained subset of all of the packages on the gallery. When uploading an extension, the curator will examine the package's dependencies, and place the package in the appropriate feed. E.g. If a dependency is `Wave=[2.0]`, the curator will put the package into the `Wave_v2.0` feed.

The curator will also try to add dependencies into the feed. However, to keep things simple, the curator only adds direct dependencies to the feed, and the package must exist on the gallery first. If a dependency isn't available, the extension installation will fail.

> **NOTE** Since a package dependency can be satisfied by a range of versions of a package, the curator has to add all compatible versions of the dependency to the feed. If transitive dependencies were handled, the curator would also have to add all versions of all packages that satisfied the dependencies of all versions of the packages that were required by the original extension, and so on. Rather than process this potentially huge number of packages (and deal with invalid dependency cycles and conflicting version numbers), for simplicity, the curator only processes direct dependencies.

The recommendation is:

* An extension should specify all transitive dependencies in its own `.nuspec` folder.
* All dependencies should be pushed to the gallery before the extension package.
* If there are any problems, either [raise an issue](https://youtrack.jetbrains.com/newIssue?project=RSRP&clearDraft=true&c=) or post to the [Google group](https://groups.google.com/forum/#!forum/resharper-plugins).
