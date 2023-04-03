[//]: # (title: Project File Language Service)

If a custom language defines a `ProjectFileType`, it must also associate that file type with a PSI language type, which ReSharper can use to build a PSI tree of the file's contents. The `IProjectFileLanguageService` interface provides this functionality, and can be considered as a bridge between the Project Model and the PSI.

The project file language service is implemented as a [per-file type component](Registration_PerLanguageComponents.md#file-type-specific-components). Given a `ProjectFileType`, it is possible to retrieve that file type's project file language service, which exposes the custom language's `PsiLanguageType`. This can be used as the key to retrieve all [per-language components](Registration_PerLanguageComponents.md#psi-language-specific-components), and get access to the custom language's lexer and parser, to build a PSI tree.

The `IProjectFileLanguageService` interface is as follows:

```csharp
public interface IProjectFileLanguageService
{
  ProjectFileType LanguageType { get; }
  IconId Icon { get; }
  PsiLanguageType GetPsiLanguageType(IProjectFile projectFile);
  IPsiSourceFileProperties GetPsiProperties(IProjectFile projectFile, IPsiSourceFile sourceFile);
  ILexerFactory GetMixedLexerFactory(ISolution solution, IBuffer buffer, IPsiSourceFile sourceFile = null);
  PreProcessingDirective[] GetPreprocessorDefines(IProject project, TargetFrameworkId targetFrameworkId);
}
```

The members are as follows:

* **`LanguageType`** - confusingly called `LanguageType`, this is the singleton instance of the derived `ProjectFileType` class that represents the [Project Model file type](ProjectFileType.md) of this language.
* **`Icon`** - the `IconId` of the icon used to represent this file. This is used whenever ReSharper needs to display the icon for a file of this type, such as in the Find Results tool window. See the section on [icons](Platform__Icons.md) for more details on icons.
* **`GetPreprocessorDefines`** - retrieves a list of preprocessor symbols that are defined for the current project, as this affects the parsing of the PSI tree. This is typically only required for compiled languages in a project where the project itself has a specific language, e.g. a C# project.

## The GetPsiLanguageType method

The **`GetPsiLanguageType`** method returns the `PsiLanguageType` derived instance that represents the custom language of the given project file. Generally speaking, this is very simple to implement, and involves returning the static instance of the PSI language type, after checking the `IProjectFile.LanguageType` is the required `ProjectFileType` (or a derived instance thereof). For example:

```csharp
public PsiLanguageType GetPsiLanguageType(IProjectFile projectFile)
{
  // Return UnknownLanguage if the actual language instance is null
  // This can happen if the language feature is disabled
  if (projectFile.LanguageType.Is<CssProjectFileType>())
    return (PsiLanguageType) CssLanguage.Instance ?? UnknownLanguage.Instance;
  return UnknownLanguage.Instance;
}
```

Note that we return `UnknownLanguage.Instance` if the actual language instance isn't available. This can happen if the language feature has been disabled.

This method can be a bit more complex. For example, the `JavaScriptProjectFileLanguageService` checks to see if the current project is a Windows JavaScript application, in which case, it returns `JavaScriptWinRTLanguage.Instance`. Otherwise it returns the standard `JavaScriptLanguage.Instance`.

## PSI source file properties

The **`GetPsiProperties`** method returns an instance of `IPsiSourceFileProperties` that defines how the given file should be handled by the PSI, such as if a PSI tree should be built for the file.

Typically, this can be an instance of, or derived from `DefaultPsiProjectFileProperties`, which provides a default implementation, given an `IProjectFile` and an `IPsiSourceFile`.

The `IPsiSourceFileProperties` interface is as follows:

```csharp
public interface IPsiSourceFileProperties
{
  bool ShouldBuildPsi { get; }
  bool IsGeneratedFile { get; }
  bool IsICacheParticipant { get; }
  bool ProvidesCodeModel { get; }
  bool IsNonUserFile { get; }
  IEnumerable<string> GetPreImportedNamespaces();
  string GetDefaultNamespace();
  ICollection<PreProcessingDirective> GetDefines();
}
```

The interface members are:

* **`ShouldBuildPsi`** - return `true` if ReSharper should build a PSI tree of this file. The default implementation will return `true` if the `ProjectFileType` is a known language.
* **`IsGeneratedFile`** - return `true` if the file is generated. ReSharper will build a PSI tree for the file, but won't run inspections. The default implementation defers to `GeneratedUtils.IsGeneratedFile`, which verifies the file name and path against the masks in the Generated Code options page.
* **`IsICacheParticipant`** - this is a flag to indicate if the caching subsystem should be notified whenever the file is changed. This is typically `true`.
* **`ProvidesCodeModel`** - return `true` if this file contributes to the semantic code model of the project. If `false`, functionality such as inspections and rename is not available. Generally, this value matches `ShouldBuildPsi`, but can be different for features such as "external sources" - decompiled or downloaded source files.
* **`IsNonUserFile`** - return `true` if the file is not a user-editable file. This is similar to `IsGeneratedFile` but does not indicate that the code is generated. Inspections and modifications are disabled when this value is `true`.
* **`GetPreImportedNamespaces`** - returns a list of namespaces that are implicitly imported. For example, `.aspx` pages can have namespaces imported by entries in the `web.config` file.
* **`GetDefaultNamespace`** - returns the default namespace of this file. Typically defined in the project settings of a CLR project. This setting is not used for non-CLR project files, e.g. JavaScript. Should return `string.Empty` if not required.
* **`GetDefines`** - returns the pre-processor symbols defined for the project. Again, this comes from the project settings, and only makes sense for compiled languages, such as C#.

## Lexer factory

The **`GetMixedLexerFactory`** method returns a factory that is used to create the lexer for a file of this type. Typically, this is just a call to the PSI language service's `GetPrimaryLexerFactory` method.

```csharp
public override ILexerFactory GetMixedLexerFactory(ISolution solution, IBuffer buffer, IPsiSourceFile sourceFile = null)
{
  var languageService = My.Instance.LanguageService();
  if (languageService != null)
    return languageService.GetPrimaryLexerFactory();
  return null;
}
```

However, if the file type is a "mixed" file, that is, it contains secondary PSI trees, such as an `.aspx` or HTML file that can contain HTML, CSS, JavaScript and even C#, this method should return a "mixed" lexer. Deriving from the `MixedProjectFileLanguageService` will provide a suitable implementation here. See the section on Secondary PSI files for more details.

## Default base classes

ReSharper provides the `ProjectFileLanguageService` and `MixedProjectFileLanguageService` abstract base classes that provide a good base implementation of `IProjectFileLanguageService`. While many of the type members are virtual, the only abstract members that need implementing are:

* `PsiLanguageType`
* `Icon`
* `GetMixedLexerFactory` - `MixedProjectFileLanguageService` provides a default implementation of this.

## Custom PSI properties provider

Some languages also implement `IProjectFileCustomPsiPropertiesProvider`, which is a per-file type component that will return a specific subclass of `ICustomPsiSourceFileProperties`.

This is also a bridge between the Project Model and the PSI, as it provides access to custom project properties, via the `GetCustomProperties<T>` extension method on `IPsiSourceFile`. For example, using this extension method on a C# file can retrieve the `ICSharpPsiSourceFileProperties` interface, which provides a `WarningsAsErrors` property. This allows PSI based inspections and analyses to set the severity level of a highlight.
