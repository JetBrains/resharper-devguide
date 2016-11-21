---
---

# Suppressing Highlights

ReSharper allows suppressing configurable highlights with specially formatted comments, such as:

```csharp
// ReSharper disable once UnassignedReadonlyField
public static readonly double? UNDEFINED_OPACITY;
```

Similarly, highlights relating to compiler warnings are not shown if the warnings are disabled in project settings, or via a `#pragma` directive. Highlights are also not shown in generated files, or code regions that should be treated like generated code, as specified in the Generated Code options page.

> **NOTE** Element Problem Analysers are run by existing daemon stages. For built in languages, ReSharper already handles filtering highlights. If you wish to host Element Problem Analysers for your own language, you will need to follow the advice here.

ReSharper will display all highlights that the daemon stage process returns for a specific file. It is up to the daemon stage process itself to perform the appropriate filtering, based on the file's language and requirements. The daemon stage can use the `FilteringHighlightingConsumer` class to perform the filtering as the results are collected during analysis.

The `IDaemonStageProcess.Execute` method expects a `DaemonStageResult` that contains a list of highlights to apply. To make it easier to capture this list, ReSharper provides a couple of implementations of the `IHighlightingConsumer` interface. This interface provides context, such as access to settings, as well as a method to consume a highlight. They also return the list of highlights to be used in `DaemonStageResult`. Typically, a daemon process will create one of these classes and pass it to the code doing the actual analysis, which will add highlights as it works.

The default implementation `DefaultHighlightingConsumer` is a simple class, and captures all highlights passed to it. This is useful for languages where filtering is not appropriate - perhaps the language doesn't support comments, or have compiler warnings.

The `FilteringHighlightingConsumer` class will check that the highlight should be shown before it's added to the list. The class will, in priority order:

* Filter out any highlights with a severity of `Severity.DO_NOT_SHOW`.
* Always include highlights with a severity of `Severity.INFO`.
* Filter out all other highlights for non-user files (such as decompiled files).
* Include highlights that correspond to compiler errors.
* Filter out all highlights in a generated file, or that are located in a generated region.
* Filter out compiler warnings that have been disabled via `#pragma` or equivalent.
* Filter highlights by specially formatted comments.

> **NOTE** The `FilteringHighlightingConsumer` class will automatically filter out highlights in generated files or regions. This means a daemon stage process, or individual highlights do not need to worry if code being analysed is generated or not - the highlights are filtered out before they are applied to the editor.

A typical daemon stage process `Execute` method that wanted to take advantage of `FilteringHighlightingConsumer` would look like this:

```csharp
public virtual void Execute(Action<DaemonStageResult> committer)
{
  var consumer = new FilteringHighlightingConsumer(this, this.SettingsStore, this.File);
  this.File.ProcessThisAndDescendants(new MyRecursiveElementProcessor(this, consumer));
}
```

This creates a new instance of `FilteringHighlightingConsumer`, passing in the daemon stage, the current settings store and the file that will be processed. The consumer is then used to collect highlights, in this case via the `MyRecursiveElementProcessor` class that is being called for all nodes in the syntax tree. This is all there is to it. Highlights are now appropriately filtered.

Any daemon stage process deriving from `CSharpIncrementalDaemonStageProcessBase` will automatically get this functionality.

## Working with custom languages

As long as a daemon stage process is for an existing language, there is no need to worry about how `FilteringHighlightingConsumer` gets the information required to correctly filter highlights. Adding a custom language would usually require adding some new per-language services that `FilteringHighlightingConsumer` will use if available.

### Generated files

The `FilteringHighlightingConsumer` class will filter out all highlights (apart from compiler errors and info highlights) that are in generated files or regions of a file that are marked as generated. A file is considered to be a generated file if it's `IPsiSourceFile.Properties.IsGeneratedFile` property returns `true`.

These file properties usually come from the current project, but can also be set via the Generated Code options page, by directory, file name or file name mask. It can also be programmatically overridden by implementing `IPsiSourceFilePropertiesProvider`.

A generated code region is provided by implementing an instance of "IFileStructureIndex".

## File structure explorers

Generated regions, comment regions and compiler warning/pragma regions are discovered by implementations of `IFileStructureExplorer`. These classes will process a file looking for comments, regions (as in `#region`) and pragma directives that describe ares of code that should have highlights filtered.

ReSharper ships with one implementation per language, but it is possible to have more than one for a particular language. Each implementation should be marked with the `[FileStructureExplorer]` attribute.

The `IFileStructureExplorer.Run` method will generate an instance of a class that derives from `IFileStructure` that lists regions of generated code, as well as block ranges for comments, and pragmas. ReSharper provides the `FileStructureBase` and `FileStructureWithRegionsBase` classes to provide a lot of functionality. A derived class will implement `Run`, and walk the file's syntax tree. If it encounters a comment node, it can pass the node and the comment text to the `FileStructureBase.ProcessComment` method, which will parse the comment text and add a generated code region, if appropriate. Similarly, preprocessor directives such as `#pragma` can pass the tree node, a list of compiler IDs and a flag to indicate if the warnings are being enabled or disabled. Finally, if the class derives from `FileStructureWithRegionBase`, it can call `ProcessStartRegion` and `ProcessEndRegion` to handle C# style `#region` nodes. The text for matching region names comes from settings - `GeneratedCodeSettingsKey.GeneratedCodeRegions`.

### Compiler errors and warnings

The `FilteringHighlightingConsumer` class will also filter out applicable compiler and error warnings. Highlights are checked for equivalent compiler IDs. If they exist, and the highlight has a severity of error, then the highlight is always displayed, regardless of regions.

Warnings are treated slightly differently. If a highlight has a compiled ID, but is a warning, it is not automatically filtered out. If the highlight is an instance of `HighlightingInfoWithOverrides` and either `OverriddenSeverity` or `OverriddenAttributeId` is set, the highlight is always shown.

Otherwise, the per-language `ICompilerWarningPreProcessor` class is used, if it exists. This will check the compiler IDs for the file, usually by checking project settings such as current warning level - for example, a highlighting that represents a level 4 warning when the project is set to level 3 will be suppressed. It will also look to see if the warning should be upgraded to an error, if the "treat warnings as errors" project setting is enabled.

ReSharper uses the `CompilerIDs` property of the `[RegisterConfigurableSeverity]` assembly attribute to specify compiler IDs for a highlight. This value can be a single ID, or a comma separated list of IDs. ReSharper will also use the `[CompiledIdForLanguage]` assembly attribute to set up a mapping between a highlight ID, a language and a compiler warning ID.
