---
---

# Registering a PSI Language

Once a custom language's [file type has been registered with the Project Model](ProjectFileType.md), the language definition needs to be registered with the PSI. If it's not registered with the PSI, ReSharper will not be able to build a PSI tree, and in fact will not consider it registered with the Project Model, either.

The PSI maintains a hierarchy of supported languages, in a similar manner to how the Project Model maintains a hierarchy of supported file types. The base class is `PsiLanguageType`, and there are again, two directly derived classes - `KnownLanguage` and `UnknownLanguage`. The first is the base class for all known languages, and can be used to mean any and all known PSI languages. The second, `UnknownLanguage`, is a sealed class, and represents a language that the PSI does not support.

A custom language should create a class that derives from `KnownLanguage`, or a suitable base class (for example, if your language is a subset of HTML, such as a `.aspx` file, you can derive from `HtmlLanguage`). The custom class should provide a programmatic name and a presentable name, and it should be marked with the `[LanguageDefinition(...)]` attribute. For example, C#:

```csharp
[LanguageDefinition(Name, Edition = ReSharperEditions.Ids.Csharp)]
public class CSharpLanguage : KnownLanguage
{
  // Must be "new" as it overrides a public field in the base
  public new const string Name = "CSHARP";

  // May need "new" if inheriting from another language, such as HTML
  // This value might be null, if the language has been disabled in Options
  [CanBeNull] public static readonly CSharpLanguage Instance;

  private CSharpLanguage() : base(Name, "C#") { }

  protected CSharpLanguage([NotNull] string name) : base(name) { }

  protected CSharpLanguage([NotNull] string name, [NotNull] string presentableName) : base(name, presentableName) { }
}
```

The `[LanguageDefinition]` attribute passes in the programmatic name. It also supports an `Edition` property, which is no longer used. This is a legacy feature to allow languages to be disabled based on which edition of the product has been purchased. This functionality has been replaced by [Zones](/Platform/Zones.md).

Each node in the PSI tree exposes its owning language via the `ITreeNode.Language` property, which returns an instance of `PsiLanguageType`.

Languages are instantiated by the `ILanguages` shell component, which will create the class using the default constructor, and set the static `Instance` field, meaning all language definitions are singletons.

The `ILanguages` interface also provides the means to retrieve and enumerate all language definitions.

```csharp
public interface ILanguages : IViewable<PsiLanguageType>
{
  IEnumerable<PsiLanguageType> All { get; }
  PsiLanguageType GetLanguageByName(string languageName);
}
```
