# Utils

<!-- Index A - Z (auto-generated. Remove this line if manually adding/removing entries) -->

<!-- toc -->
<!-- toc stop -->

## M

### ModuleQualificationUtil

<!-- Begin ModuleQualificationUtil -->

```cs
public static class ModuleQualificationUtil
{
  public static string GetModuleName(IPsiModule psiModule);
  public static string GetModuleQualification(IPsiModule module, IModuleReferenceResolveContext context);
  public static string GetTypeName(IXmlAttributeValue value, out AssemblyNameInfo assemblyName);
  public static TreeTextRange GetTypeNameCompletionRange(string text, TreeOffset offset);
  public static TextRange GetTypeNameFullRange(string text, int offset);

  public static bool IsIdentifierChar(Char c);
  public static bool IsTypeNameChar(Char c);

  public static ICollection<ITypeElement> ResolveType(string clrName, IPsiModule module, bool caseSensitive, IModuleReferenceResolveContext context);
}
```

<!-- End ModuleQualificationUtil -->

Helper methods to support references to CLR types inside XML documents, e.g. web.config files.

* `GetModuleName` returns the name of the given module, e.g. if it's a project, then return the assembly name, else return the module's name.
* `GetModuleQualification` returns the fully qualified assembly name of the module, either the compiled assemblies name, or the name built up from the project settings.
* `GetTypeName` returns a CLR type name and assembly name info from an assembly qualified type name specified in an XML attribute. Defers to `JetBrains.Metadata.Utils.ModuleQualificationUtil.GetTypeName`.
* `GetTypeNameCompletionRange` takes in a string and a starting offset and works out from the text where a type name will start and end, by walking backwards from the start while `IsTypeNameChar` is true, and walking forwards while `IsIdentifierChar` is true.
* `GetTypeNameFullRange` returns the full range of a type name inside a given string. This is different to `GetTypeNameCompletionRange` by allowing type name characters after the start offset, which allows for the current offset to be in a namespace, or for the type name to include nested types.
* `IsIdentifierChar` returns `true` if the given character is a letter, digit or the underscore symbol (`_`).
* `IsTypeNameChar` returns `true` if the given character is a letter, digit, or either an underscore, period, plus or backtick symbol (`_`, `.`, `+` or `\``).
* `ResolveType` retrieves a collection of `ITypeElement` instances for the given CLR name, using an `ISymbolScope` retrieved via `IPsiServices.Symbols`.

## R

### ReferenceWithTokenUtil

<!-- Begin ReferenceWithTokenUtil -->

```cs
public static class ReferenceWithTokenUtil
{
  public static void AddRestoreTransactionAction(IPsiServices psiServices, IReferenceWithToken referenceWithToken, ElementRange<IXmlToken> oldRange);

  public static IXmlToken SetText(IXmlToken token, TreeTextRange oldRange, string newText, ITreeNode elementToDropReferences);
  public static IReferenceWithToken SetText(IReferenceWithToken reference, string newText);
}
```

<!-- End ReferenceWithTokenUtil -->

Helper methods for handling references within a token node. The methods defer to `ReferenceWithinElementUtil<IXmlToken>`. The `SetText` methods provide a factory method to create an appropriate token to replace the existing element.

## X

### XmlAttributeUtil

<!-- Begin XmlAttributeUtil -->

```cs
public static class XmlAttributeUtil
{
  public static void SetValue(IXmlAttribute attribute, string unquotedValue);

  public static DocumentRange UnquotedValueDocumentRange(IXmlAttribute attribute);
  public static TreeTextRange UnquotedValueRange(IXmlAttribute attribute);
}
```

<!-- End XmlAttributeUtil -->

Helper methods for working with XML attributes. `SetValue` replaces an attributes value by creating a new instance of `IXmlAttributeValue` and modifying the tree (it takes the write lock, so the caller doesn't have to). The other methods return the range of the unquoted value of the attribute, relative to the tree, and relative to the document. These may be different if the XML `IFile` is not the primary PSI tree in the file.

### XmlReferenceUtil

<!-- Begin XmlReferenceUtil -->

```cs
public static class XmlReferenceUtil
{
  public static TReference FindReferenceRecursively<TReference>(ITreeNode element, Predicate<TReference> predicate);
}
```

<!-- End XmlReferenceUtil -->

Helper method that will recursively walk down the PSI tree starting at `element`, looking for a particular reference, as decided by `predicate`.

### XmlTagUtil

<!-- Begin XmlTagUtil -->

```cs
public static class XmlTagUtil
{
  public static bool CanBeEmptyTag(IXmlTag tag);

  public static void MakeCompound(IXmlTag tag);
  public static void MakeEmptyTag(IXmlTag tag);
}
```

<!-- End XmlTagUtil -->

Helper methods for working with `IXmlTag`.

* `CanBeEmptyTag` returns `true` if the given tag doesn't have any significant children (e.g. anything other than whitespace), and checks that the current XML language supports converting this tag to be empty, by calling `IXmlLanguageSupport.CanMakeTagEmpty`.
* `MakeCompound` converts an empty, self-closed tag to an empty tag with a header and footer, (e.g. `<foo></foo>`). Modifies the tree, taking the write lock.
* `MakeEmpty` removes all inner nodes and the footer, and converts the header into a self-closing tag (e.g. `<foo />`). Modifies the tree, taking the write lock.

### XPathUtil

<!-- Begin XPathUtil -->

```cs
public static class XPathUtil
{
  public static IList<T> GetNestedTags<T>(IXmlTagContainer container, string xpath);
}
```

<!-- End XPathUtil -->

Walks the child tags of the `container` until it has satisfied the given XPath-like expression. The `xpath` parameter isn't really an XPath string, but XPath-like. The path must be relative to the current `container`, and walks down the tree comparing the tag names at each level with the current path segment name. If the name matches, or the path segment is the wildcard symbol `*`, the walk continues. The tags that match the last path segment name are returned as part of the list.


