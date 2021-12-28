[//]: # (title: Registering a Project File Type)

If a custom language is to be the primary language of a certain file, it should register a file type with the Project Model.

 >  Not all custom languages require a file type. For example, regular expressions are handled as an injected PSI language. The string literal that describes the regular expression in, say, a C# file, becomes a holder for a new PSI tree that describes the content of the regular expression. It doesn't make sense for a file to be a "regular expression file", so there is no project file type for regular expressions. Instead, the host language, such as C#, VB or JavaScript explicitly creates the injected PSI language.
 >
 {type="note"}

Every file that is included in a project in the Project Model exposes its file type via the `IProjectFile.LanguageType` property. This is an instance of a subclass of `ProjectFileType`, which is the root class in a hierarchy of classes that represent file types.

There are two classes that directly derive from `ProjectFileType`. The first is `KnownFileType`, which is an abstract class that represents a file type that ReSharper knows about, and is the base class for all known file types. It can be used instead of an explicit file type to indicate any or all known file types. The second is `UnknownFileType`, which is a sealed class, and only used when a file doesn't have a known file type.

A new file type for a custom language should derive from `KnownProjectFileType`, and specify a programmatic name, a presentable name and the file extensions that identify the file.

For example, here is the HTML file type:

```csharp
[ProjectFileTypeDefinition(Name)]
public class HtmlProjectFileType : KnownProjectFileType
{
  // These need to be "new" as they are also defined in the base class
  public new const string Name = "HTML";
  public new static readonly HtmlProjectFileType Instance;

  private HtmlProjectFileType() : base(Name, "Html", new[] {HTML_EXTENSION, HTM_EXTENSION})
  {
  }

  protected HtmlProjectFileType(string name) : base(name)
  {
  }

  protected HtmlProjectFileType(string name, string presentableName) : base(name, presentableName)
  {
  }

  protected HtmlProjectFileType(string name, string presentableName, IEnumerable<string> extensions) : base(name, presentableName, extensions)
  {
  }

  public const string HTML_EXTENSION = ".html";
  public const string HTM_EXTENSION = ".htm";
}
```

Note that this file type registers two file extensions - `.html` and `.htm`. The programmatic name is `"HTML"`, which is also available as a public constant, should anyone else need to identify the file type by name. The presentable name is `"Html"`. (For clarity the C# file type has a programmatic name of `CSHARP` and a presentable name of `C#`.)

Each class is marked with the `[ProjectFileTypeDefinition]` attribute, which is a [Component Model part attribute](ContainersPartsCatalogues.md) used by the implementation of `IProjectFileTypes` to find and instantiate the file type. This is not a standard Component Model attribute such as `[ShellComponent]` or `[SolutionComponent]`. The lifecycle is different, and only the default constructor is used - injection is not supported. Furthermore, when the `IProjectFileTypes` implementation creates an instance of the file type, it also sets the static `Instance` field, which means the file types are singletons.

The HTML file type above has more than one constructor. Typically, these aren't used, but are available for derived file types. For example, the `AspProjectFileType`, which provides a file type for `.aspx`, `.ascx`, `.master` and `.asax` files, derives from `HtmlProjectFileType` (and uses one of the protected constructors to override the `HTML` name). Deriving from another file type creates an "is a" relationship, and the deriving file type "inherits" traits and services of the base file type. So, a `.aspx` file "is a" HTML file, and any services that are targeted at HTML files also apply to `.aspx` files (but not vice versa). Similarly, the `TypeScriptProjectFileType` is a subclass of `JavaScriptProjectFileType`, which makes sense, as TypeScript is a superset of JavaScript.

## File extension mapping

A file's project file type can be fetched via its file extension, by using the `IProjectFileExtensions` shell component.

```csharp
public interface IProjectFileExtensions
{
  ProjectFileType GetFileType(string extension);
  IEnumerable<string> GetExtensions(ProjectFileType fileType);
  ISimpleSignal Changed { get; }
}
```

This interface allows getting the file type from an extension, including the dot (.e.g `.cs`). It will also get the list of extensions supported by a file type, and raise a signal if this information changes.

The information for the file extension mapping comes from various `IFileExtensionMapping` shell components. The default implementation uses the extensions given in the `ProjectFileType` instances, but ReSharper also provides an implementation that retrieves information from Visual Studio. It fetches the list of editors that Visual Studio knows about, and the file extensions registered with each editor. It then uses any `IVsEditorGuidToFileTypeConverter` shell components to convert an editor's `GUID` to a `ProjectFileType`. This allows ReSharper to correctly handle files that have a non-standard file extension, but have been opened in a specific editor in Visual Studio. The `IFileExtensoinMapping` interface might also be useful if adding support for a language that Visual Studio already supports.