---
---

# Per-language Components

The PSI is designed to work with multiple languages. It supports many features, from parsing and code rewriting, to formatting and navigation. The PSI provides cross language infrastructure to support these features, but each feature requires language specific implementations, for example, the `LanguageService` abstract base class provides the interface to create lexers and parsers for a particular file type, but the implementation of the class must be specific to each language.

As such, the normal `[ShellComponent]` and `[SolutionComponent]` components do not help here. These component attributes can be used to create multiple implementations of an interface or abstract base class, but consumers receive a list of instances, while the PSI requires a single instance for a specific language.

Furthermore, languages are hierarchical, allowing for more specific or perhaps fallback implementations of services. For example, XAML inherits from the XML language, so could provide its own specific code formatter, or fall back to the XML formatter.

The PSI needs a mechanism that will return a single implementation of an interface or base class for a given language type. It handles this with a [custom component container](/Platform/ComponentModel/ContainersPartsCatalogues.md).

The PSI defines the `[Language]` attribute that derives from `PartAttribute`, in the same way that `ShellComponentAttribute` and `SolutionComponentAttribute` do. Since it derives directly from `PartAttribute`, it doesn't get included in either the shell or solution component container. Instead, the `ILanguageManager` component maintains a custom container (strictly speaking, it does this through inheritance, so it *is* a custom container) that filters parts in the product catalogue sets by the `LanguageAttribute`.

The component specifies the language it relates to by passing the type in as a parameter to the constructor:

```csharp
[Language(typeof(CSharpLanguage))]
public class CSharpCodeFormatter : ICodeFormatter
{
}
```

The language type is used as a key for lookup, using the `ILanguageManager` methods, where `T` is the type of the service being retrieved, and `TLanguage` is the type of the language. Alternatively, the language is specified as an instance of the language type, perhaps retrieved via `ITreeNode.Language`:

```csharp
public interface ILanguageManager
{
  T GetService<T, TLanguage>()
    where T : class
    where TLanguage : PsiLanguageType;
  T TryGetService<T, TLanguage>()
    where T : class
    where TLanguage : PsiLanguageType;
  T GetService<T>(PsiLanguageType languageType)
    where T : class;
  T TryGetService<T>(PsiLanguageType languageType)
    where T : class;
  IEnumerable<T> GetServicesFromAll<T>()
    where T : class;
  IEnumerable<T> GetMultipleServicesFromAll<T>()
    where T : class;
  IEnumerable<T> GetServices<T>(PsiLanguageType languageType)
    where T : class;
  bool HasService<T>(PsiLanguageType languageType)
    where T : class;
  T TryGetCachedService<T>(PsiLanguageType languageType)
    where T : class;
}

[SolutionComponent]
public class Foo
{
  public Foo(Lifetime lifetime, ILanguageManager languageManager)
  {
    var formatter = languageManager.GetService<ICodeFormatter, CSharpLanguage>();
    // ...
  }
}
```

The instance of `ILanguageManager` can be retrieved via normal shell component rules, either by constructor injection, or by using `Shell.Instance.GetComponent<ILanguageManager>()`.

## Lifetime

The `ILanguageManager` component is decorated with the `[PsiSharedComponent]` attribute, which derives from `ShellComponentAttribute`, which means that `ILanguageManager` is a normal shell component, and has the normal lifetime of a shell component. When the `ILanguageManager`'s `Lifetime` terminates, all of the parts it manages are also terminated.

The use of the `[PsiSharedComponent]` attribute is not significant in and of itself. Its only significance is to indicate that this component is part of the implementation of the PSI, and instances are shared across solutions. As such, it can simply inherit from `[ShellComponent]` to get the desired behaviour.

