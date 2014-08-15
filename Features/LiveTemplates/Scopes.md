# Scopes

ReSharper's Live Templates can declare the _scope_ in which they are available. That is, instead of allowing all templates to be inserted at any location in any file, a template can be constrained to certain file types, and even to certain locations within those files. For example, a template can be constrained not just to C# files, but to any location where a namespace declaration, type definition, or an expression is valid.

Furthermore, the template scope can be parameterised. For example, a C# template can state that it is only available in C# 3.0 files (perhaps the template wants to make use of LINQ), or in files that match a specific file pattern (e.g. "actions.xml", "*.html", etc.)

A template can have multiple scope points, in which case, the template is available if any of the scope points match the current context.

<!-- TODO: Add image for the scope UI -->

In this way, templates can be constrained to be more targeted and useful, even allowing multiple templates with the same shortcut, as long as they are valid in different scopes.

## Implementing a custom scope provider

The `TemplateScopeManager` shell component is the main entry point for working with scope points, and it maintains a list of `IScopeProvider` shell components. As such, it can be extended by a class implementing `IScopeProvider` and decorated with the `[ShellComponent]` attribute.

> **Note** The list of `IScopeProvider` components maintained by the `TemplateScopeManager` is injected as an `IEnumerable<IScopeProvider>` rather than an `IViewable<IScopeProvider>`. Therefore, the list does not update when new components are made available. Live loading of plugins is not supported; when a new plugin is installed, ReSharper must be restarted before the new scope provider is used.

The `IScopeProvider` interface declares three methods that are used to create scope points:

```cs
public interface IScopeProvider
{
  IEnumerable<ITemplateScopePoint> ProvideScopePoints(TemplateAcceptanceContext context);
  ITemplateScopePoint ReadFromXml([NotNull] XmlElement scopeElement);
  ITemplateScopePoint CreateScope(Guid scopeGuid, string typeName, IEnumerable<Pair<string, string>> customProperties);
}
```

* **`ProvideScopePoints`** - returns all scope points that are valid at the context described by the given `TemplateAcceptanceContext` instance, which provides details about file and text caret position. Parameterised scope points are either set to a reasonable value (for example, C# scope points can set a language version of 3.0 when the current context is in a LINQ query), or replaced by a scope point that makes more sense for the current context (instead of creating a file mask scope point of "\*.\*", a project file scope point is returned. This scope point knows how to compare itself against a file mask scope point used in a template)
* **`ReadFromXml`** - used to read a scope point from an XML file. This is not typically used any more, as recent versions of ReSharper store and export templates in `.dotSettings` format, rather than the proprietary XML read by this method. However, the built-in providers still support this, in order to upgrade existing files in the old format. The complement to this method is the `ITemplateScopePoint.WriteToXml` method, which is still supported, but no longer exposed by ReSharper.

    The `ScopeProvider` abstract base class used by all of the built-in providers will create a new instance based on the short type name in the `type` attribute of the XML element. Deriving classes should call the base method to create the instance, then populate it based on other attributes or child elements of the given XML element.

* **`CreateScope`** should create an instance of `ITemplateScopePoint` based on the given short type name. It receives a GUID identifier that is unique for each template scope point instance (remember, scope points might be parameterised) and custom properties as string name/value pairs. This method is used to read the scope point from a `.dotSettings` file, which stores all information as name/value pairs.

    Again, the `ScopeProvider` abstract base class will create a new instance based on the given type name and populate the `ITemplateScopePoint.UID` with the unique ID. Derived classes should call the base class and populate the new instance based on the custom properties.

The `ScopeProvider` abstract base class provides a mechanism for creating specific instances of `ITemplateScopePoint` based on a given type name. While it can simply instantiate the type based on the type name, it defers to a list of creator functions that take in the type name and create the scope point instance. This provides an extension point for custom creation, while also making it easy to create simple instances.

For example, the CSS scope provider will set up a list of creator functions that simply defer to `ScopeProvider.TryToCreate<T>` which will create the type if the type name matches:

```cs
public class CssScopeProvider : ScopeProvider
{
  public CssScopeProvider()
  {
    Creators.AddRange(new List<Func<string, ITemplateScopePoint>>
    {
      s => TryToCreate<InCssFile>(s),
      s => TryToCreate<InCssExpression>(s),
      s => TryToCreate<InCssStatement>(s),
    });
  }

  // ...
}
```

### Example IScopeProvider.ProvideScopePoints implementation

The following is an example of an implementation of `IScopeProvider.ProvideScopePoints` for CSS files. It examines the current context, as provided by `TemplateAcceptanceContext` and returns scope points that are valid:

```cs
public override IEnumerable<ITemplateScopePoint> ProvideScopePoints(TemplateAcceptanceContext context)
{
  // Ensure we have an IPsiSourceFile, a valid document range, and the file is a CSS file
  if (context.SourceFile == null)
    yield break;

  var documentRange = new DocumentRange(context.Document, context.CaretOffset);
  if (!documentRange.IsValid())
    yield break;

  var file = context.SourceFile.GetPsiFile<CssLanguage>(documentRange);
  if (file == null)
    yield break;

  // Get the prefix at the text caret. This is treated as the shortcut that invokes the template
  // Then get the AST element that represents the prefix
  var prefix = LiveTemplatesManager.GetPrefix(context.Document, context.CaretOffset);
  var element = file.FindTokenAt(context.Document, context.CaretOffset - prefix.Length) as ITokenNode;

  if (element == null && String.IsNullOrEmpty(prefix))
  {
    if (context.Document.GetTextLength() == 0)
    {
      // There is no element at the caret, no known prefix and the document is empty
      // Valid scope points are anywhere in a CSS file, and anywhere a CSS statement is valid
      yield return new InCssFile();
      yield return new InCssStatement();
      yield break;
    }

    // If the caret is at the end of the file, get the token just before the EOF
    if (context.Document.GetTextLength() == context.CaretOffset)
      element = file.FindTokenAt(context.Document, context.CaretOffset - 1) as ITokenNode;
  }

  // There is no token, so no valid scope points
  if(element == null)
    yield break;

  var treeOffset = file.Translate(context.Document, context.CaretOffset);
  if (!treeOffset.IsValid())
    yield break;

  // Return a valid CSS file scope point
  yield return new InCssFile();

  // Look at the containing node, and return either a CSS expression or CSS statement scope point
  if (element.GetContainingNode<ICssPropertyValue>(true) != null || element.GetContainingNode<ICssExpression>(true) != null)
    yield return new InCssExpression();
  else if (element.GetContainingNode<ICssPropertyStatement>(true).IfNotNull(s => s.Property.IfNotNull(p => p.ColonToken.IfNotNull(n => n.GetTreeStartOffset() < element.GetTreeStartOffset()))))
    yield return new InCssExpression();
  else if (element.GetContainingNode<IMediaFeature>(true).IfNotNull(f => f.ColonToken.IfNotNull(n => n.GetTreeStartOffset() < element.GetTreeStartOffset())) )
    yield return new InCssExpression();
  else if(element.GetContainingNode<ICssPropertyStatement>(true) == null && element.GetContainingNode<IMediaFeature>(true) == null)
  {
    if (element.GetContainingNode<ICssBlock>() != null || element.GetContainingNode<IMediaQueryList>() != null)
      yield return new InCssStatement();
  }
}
```

The call to `LiveTemplatesManager.GetPrefix` gets the "word" that precedes the current text caret position. It is used here to find the position in the document that marks the start of the context of the scope point. This position is used to retrieve the node from the abstract syntax tree, which can then be used to see if the scope point is inside an expression or a statement, or so on.

A "word" is defined as any character preceding the current text caret that is a letter or digit, or is one of a set of allowed characters. The overload shown above does not specify any additional characters, so uses the default of the underscore character `'_'`. Another overload takes an array of allowed chars.

```cs
public static string GetPrefix(IDocument document, int caretOffset, params char[] allowedChars)
```

For example, the `HTMLScopeProvider` class calls `GetPrefix` with an array of `{ '_' }` when fetching the prefix for file scope, and uses `{ '_', '<', ':' }` when fetching the prefix for tag scope.

## Implementing a custom scope point

A scope point is implemented by deriving from `ITemplateScopePoint`, and is usually implemented by deriving from the `TemplateScopePoint` abstract base class. It is not a component from the Component Model, but just a class created by an `IScopeProvider`. The interface is:

```cs
public interface ITemplateScopePoint
{
  bool IsSubsetOf(ITemplateScopePoint other);
  string Prefix { get; }
  string CalcPrefix(IDocument document, int caretOffset);
  [NotNull] XmlElement WriteToXml(XmlElement element);
  string PresentableShortName { get; }
  PsiLanguageType RelatedLanguage { get; }
  Guid UID { get; set; }
  Guid GetDefaultUID();
  string GetTagName();
  IEnumerable<Pair<string, string>>  EnumerateCustomProperties();
}
```

* **`IsSubsetOf`**
* **`Prefix`**
* **`CalcPrefix`**
* **`WriteToXml`** - used to write the scope point, including parameters, to XML. This is the complement to `IScopeProvider.ReadFromXml`. ReSharper no longer uses this method for saving scope points, as templates are now saved in `.dotSettings` format.
* **`PresentableShortName`** - a short description of the scope point, displayed in the template editor UI. Can be a static string, or reflecting the parameters, such as `"'*.*; *.html' files"`.
* **`RelatedLanguage`** - returns the `PsiLanguageType` that applies to this scope point. If the scope point is language agnostic, the default implementation from `TemplateScopePoint.RelatedLanguage` will return `null`.
* **`UID`** - a property containing the unique ID for this instance of the scope point. Automatically set when creating the scope via `IScopeProvider.CreateScope`.
* **`GetDefaultUID`**
* **`GetTagName`** - returns the short type name, saved to the `.dotSettings` file, and used to pass to `IScopeProvider.CreateScope`. The default implementation in the `TemplateScopePoint` returns the short type name.
* **`EnumerateCustomProperties`** - returns a set of name/value string pairs listing the custom properties of the scope point. Used to save the properties to the `.dotSettings` file format. The `TemplateScopePoint` abstract base class returns an empty enumeration.



`IMainScopePoint` used by QuickListSupport, IScopeCategoryUIProvider and SupportedQuickList
