# SDK Reference

> **Warning** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

The ReSharper SDK is a set of Visual Studio project and item templates designed to help you create and build ReSharper plugins. You can download it [here](http://www.jetbrains.com/resharper/download/).

## What’s in the SDK

The SDK installs **Visual Studio project and item templates**. The project templates are for quickly creating projects for your plugins and for plugin tests. The item templates are for adding skeleton implementations of common extension points to an existing project.

The project templates also include a **copy of the SDK and test NuGet packages**, which are automatically added as NuGet references to the created projects.

The SDK also installs some **sample plugins** into the `Program Files\JetBrains\ReSharper\v8.2\SDK\Samples` folder. Since this folder lives under `Program Files`, the files cannot be edited here. To use the samples, you must copy the folders to a writable location.

## Getting Started

To get a feel for the way in which plugins are developed, make a writable copy of the `SamplePlugin` folder from the `SDK\Samples`. This project illustrates the following ReSharper features:

* A code cleanup module
* A context action
* A daemon stage
* An element problem analyzer
* A quick-fix

Of course, the SDK itself is not limited to just these extension points; there are many more available.

Once you’ve familiarized yourself with the sample project, go ahead and create one of your own! The project templates for ReSharper plugins are available for both Visual Basic and Visual C#. Corresponding test project templates are also available.

> **Info** ReSharper plugin development is a lot easier if you use the tools such as the Psi Viewer which are available in ReSharper’s Internal Mode. To enable Internal Mode, edit Visual Studio’s shortcut, adding the string `/ReSharper.Internal` to the end. Restart Visual Studio to see an additional Internal menu under the ReSharper top-level menu.

## Samples

The SDK comes with three sets of samples, all of which demonstrate the different aspects of ReSharper that can be extended.

First, in the `SamplePlugin` folder, we have a plugin that shows how a single feature - the option to convert `int.MaxValue` integer literals into correspondingly named constants. The `SamplePlugin` shows how to implement the following features:

* Action
* Context Action
* Quick-Fix
* Daemon Stage
* Problem Analyzer
* Code Cleanup
* Options Page
* Settings

as well as the associated tests.

> **Info** Unfortunately, the `SamplePlugin` project that ships with the 8.2 SDK expects the `packages` folder to be in the folder *above* the location of the solution file. NuGet's package restore will restore the packages to the `packages` folder at the same location as the solution.
>
> To fix this, please replace the two `.csproj` files with these: [SamplePlugin.csproj](SamplePlugin.csproj) and [SamplePlugin.Tests.csproj](SamplePlugin.Tests.csproj). They are the same as the existing `.csproj` files, except any references to `..\..\packages\...` have been replaced with `..\packages\...`
>
> This will be fixed in a future update to the SDK.

The second set of examples is the ReSharper PowerToys. The PowerToys contain several projects illustrating different features of ReSharper extensibility, as shown in the table below.

* **CyclomaticComplexity:** Daemon Stage, Highlighting, Options Page, Settings
* **ExploreTypeInterface:** Action, Tool Window
* **FindText:** Action, Search
* **GenerateDispose:** Generator
* **Gist:** Action, Settings, OptionsPage
* **LiveTemplatesMacro:** Live Template Macro
* **MakeMethodGeneric:** Refactoring
* **MenuItem:** Action
* **OptionsPage:** Settings, Options Page
* **Xml and Html:** Context Action
* **ZenCoding:** Action, Options Page

The third set of examples involves the Psi language plugin contained in the `PsiPlugin` and `PsiPluginTest` folders. This set of examples illustrate the concepts used in developing support for new languages for ReSharper. Further details are outlined in the following section.

## SDK NuGet package

The SDK NuGet package bundles not just the assemblies required to build a plugin, but also MSBuild targets files and tools. If you look in the `JetBrains.ReSharper.SDK` package folder, you'll see the following folders:

* **bin** - the SDK NuGet package doesn't use a lib folder, since NuGet will automatically add these files as direct references in your .csproj file. The major downside of this is that it's not possible to set `CopyLocal` to `False` on these files, so they would always get (unnecessarily) copied to the output directory. Instead, the SDK NuGet package stores the assemblies in the `bin` folder, and adds references to them with targets defined in the `build` folder.
* **build** - contains MSBuild targets and props files to add references to the assemblies in the bin folder, setting `CopyLocal` to `False` for a plugin project, and `True` for test projects. It also sets up custom tasks and ensures several native dlls are copied to the output directory for tests.
* **tools** - contains two further folders, `MSBuild` and `PsiGen`
* **tools\MSBuild** - contains various custom MSBuild tasks and targets, such as compiling icons to identifiers
* **tools\PsiGen** - contains several tools, related to adding PSI language support to plugins. It includes a token generator, a lexer (CsLex) and a custom parser creator, which converts .psi files to parser classes. These are demonstrated in the `PsiPlugin` sample.

## Updating Existing Projects

If you have an existing project that already targets a previous version of ReSharper, you can update to the latest SDK by simply updating the NuGet dependencies. E.g. assuming you were targeting ReSharper 8.1, you could update to support ReSharper 8.2 by updating the SDK NuGet package from 8.1.555 to 8.2.1158.

## Running Your Project

To get your plugin running inside Visual Studio, you need to tell ReSharper to load it. You can use the `/ReSharper.Plugin` command line argument for this, passing in the path to your plugin's dll. Something like:

    devenv.exe /ReSharper.Plugin MyPlugin.dll

Once ReSharper has loaded, you can look at the Plugins page in the Options dialog to see if it's loaded correctly. You can expand the "Show Details" and "Show Additional Developer Information" panels to get more information about how ReSharper is loading your plugin.

Note that you can't load a plugin on the command line that is already loaded! You must ensure that the plugin isn't installed locally (e.g. `%LOCALAPPDATA%\JetBrains\ReSharper\v8.2\plugins`) or installed as part of an extension. Please uninstall before trying to debug your plugin.

The SDK project templates set the debug command line options for this by default.

## Testing Your Project

ReSharper contains project templates to create test projects. These projects add a reference to both the `JetBrains.ReSharper.SDK` and the `JetBrains.ReSharper.SDK.Tests` NuGet packages.

ReSharper's tests are acceptance tests. They create an in-memory ReSharper environment, and open a project and process a file. The result of the process is stored in a file and compared against a "gold" file. If the files are the same, the test passes. If not, the test fails.

The location of the test folder is determined by walking up the directory structure from the test assembly looking for a `test\data` path. Conventionally, the solution directory contains two folders - `src` and `test`. The `src` folder contains the plugin project and source, the `test` folder contains a `src` folder and a `data` folder for the test project and data, respectively.

Test files also support control identifiers. For example, to indicate the placement of the caret, one can use the `{caret}` identifier right in the input/gold files.

## Continuous Integration

The SDK NuGet packages are intended to be used with NuGet's package restore feature. You do not need to commit your `packages` folder to source control.

## Where to Get Help

If you have a problem, don’t hesitate to contact us - we are always happy to help! You can write a comment under one of these Confluence pages, write an SDK-related feature requires in our [YouTrack tracker](http://youtrack.jetbrains.net/dashboard), tweet us [@resharper](http://twitter.com/resharper) or [@jetbrains](http://twitter.com/jetbrains) or leave a message at the [ReSharper Discussion Forums](http://devnet.jetbrains.net/community/resharper/resharper_community?view=discussions). The choice is yours!

