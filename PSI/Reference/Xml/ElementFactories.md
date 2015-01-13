# ElementFactories

<!-- Not auto-indexed, so we can split definitions between XML and XML DTD.
     Also not including ClrDocCommentElementFactoryImpl + DocCommentElementFactory.
     These are XML Doc Comment PSI element factories -->

The XML PSI has several classes related to element factories, that is, for creating [`IXmlTreeNode`](TreeNodes.md#ixmltreenode) derived instances. Some of the classes are for internal use, or implementation details. They are documented here for completeness, and to avoid confusion when spelunking around the code in dotPeek.

The [`XmlElementFactory`](#xmlelementfactory) class is the class to use for creating tags, attributes or an entire [`IXmlFile`](#ixmlfile) instance. Use the `XmlElementFactory.GetInstance` method to get a copy of the class. The [`IXmlElementFactory`](#ixmlelementfactory) interface is a lower level API which can create any [`IXmlTreeNode`](TreeNodes.md#ixmltreenode) instance. The default implementation is [`XmlTreeNodeFactory`](#xmltreenodefactory) and can be retrieved using the `XmlTreeNodeFactory.GetInstance` method.

All other classes and interfaces are used internally, and will probably not be used from client code.

<!-- toc -->
<!-- toc stop -->

## D

### DefaultXmlElementFactoryContext

<!-- Begin DefaultXmlElementFactoryContext -->

```cs
public class DefaultXmlElementFactoryContext :
  IXmlElementFactoryContext
{
  public static DefaultXmlElementFactoryContext Instance;

  public T GetContainingNode<T>(ITreeNode currentNode, bool returnCurrent);
  public IXmlTag GetParentTag(IXmlTag currentTag);
}
```

<!-- End DefaultXmlElementFactoryContext -->

Default implementation of [`IXmlElementFactoryContext`](#ixmlelementfactorycontext) that defers to `ITreeNode.GetContainingNode<T>()` and `XmlTagNavigator.GetByTag()`.

### DelegatingXmlElementFactory

<!-- Begin DelegatingXmlElementFactory -->

```cs
public class DelegatingXmlElementFactory :
  IXmlElementFactory
{
  public DelegatingXmlElementFactory(IXmlElementFactory factory);

  IXmlElementFactory Factory { get; set; }
  PsiLanguageType LanguageType { get; }
  XmlElementTypes XmlElementType { get; }
  XmlTokenTypes XmlTokenTypes { get; }

  public IAnyContent CreateAnyContent();
  public IAttDef CreateAttDef();
  public IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlAttributeValue CreateAttributeValue(XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  public IAttType CreateAttType();
  public IXmlCData CreateCData();
  public IXmlComment CreateComment();
  public IDocTypeDeclaration CreateDocTypeDeclaration();
  public IDTDAttListDecl CreateDTDAttListDecl();
  public IDTDElementDecl CreateDTDElementDecl();
  public IDTDEntityDecl CreateDTDEntityDecl();
  public IDTDNotationDecl CreateDTDNotationDecl();
  public IEmptyContent CreateEmptyContent();
  public IXmlSyntaxErrorElement CreateError(XmlSyntaxErrorType errorType);
  public IXmlFile CreateFile();
  public IGrouppedContent CreateGrouppedContent();
  public IXmlIdentifier CreateIdentifier(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  public IDTDBody CreateIntSubset();
  public IDTDNDataDecl CreateNDataDecl();
  public IProcessingInstruction CreatePI();
  public IXmlProcessingInstruction CreatePIXml();
  public IExternalId CreatePublicExternalID();
  public IRepetitionContent CreateRepetionContent();
  public IXmlTag CreateRootTag(IXmlTagHeader header, IXmlElementFactoryContext context);
  public IElementContent CreateSimpleContent();
  public IExternalId CreateSystemExternalID();
  public IXmlTag CreateTag(IXmlTagHeader header, IXmlTag parentTag, IXmlElementFactoryContext context);
  public IXmlTagFooter CreateTagFooter(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlTagHeader CreateTagHeader(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlToken CreateToken(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
}
```

<!-- End DelegatingXmlElementFactory -->

## I

### IXmlElementFactory

<!-- Begin IXmlElementFactory -->

```cs
public interface IXmlElementFactory
{
  PsiLanguageType LanguageType { get; }
  XmlElementTypes XmlElementType { get; }
  XmlTokenTypes XmlTokenTypes { get; }

  IAnyContent CreateAnyContent();
  IAttDef CreateAttDef();
  IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  IXmlAttributeValue CreateAttributeValue(XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  IAttType CreateAttType();
  IXmlCData CreateCData();
  IXmlComment CreateComment();
  IDocTypeDeclaration CreateDocTypeDeclaration();
  IDTDAttListDecl CreateDTDAttListDecl();
  IDTDElementDecl CreateDTDElementDecl();
  IDTDEntityDecl CreateDTDEntityDecl();
  IDTDNotationDecl CreateDTDNotationDecl();
  IEmptyContent CreateEmptyContent();
  IXmlSyntaxErrorElement CreateError(XmlSyntaxErrorType errorType);
  IXmlFile CreateFile();
  IGrouppedContent CreateGrouppedContent();
  IXmlIdentifier CreateIdentifier(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  IDTDBody CreateIntSubset();
  IDTDNDataDecl CreateNDataDecl();
  IProcessingInstruction CreatePI();
  IXmlProcessingInstruction CreatePIXml();
  IExternalId CreatePublicExternalID();
  IRepetitionContent CreateRepetionContent();
  IXmlTag CreateRootTag(IXmlTagHeader header, IXmlElementFactoryContext context);
  IElementContent CreateSimpleContent();
  IExternalId CreateSystemExternalID();
  IXmlTag CreateTag(IXmlTagHeader header, IXmlTag parentTag, IXmlElementFactoryContext context);
  IXmlTagFooter CreateTagFooter(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  IXmlTagHeader CreateTagHeader(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  IXmlToken CreateToken(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
}
```

<!-- End IXmlElementFactory -->

### IXmlElementFactoryContext

<!-- Begin IXmlElementFactoryContext -->

```cs
public interface IXmlElementFactoryContext
{
  T GetContainingNode<T>(ITreeNode currentNode, bool returnCurrent);
  IXmlTag GetParentTag(IXmlTag currentTag);
}
```

<!-- End IXmlElementFactoryContext -->

Helper methods used while parsing XML into [`IXmlTreeNode`](TreeNodes.md#ixmltreenode) instances. Allows treating a node as though it has a different parent or containing node while parsing.

## R

### ResyncXmlElementFactory

<!-- Begin ResyncXmlElementFactory -->

```cs
public class ResyncXmlElementFactory :
  DelegatingXmlElementFactory
{
  public ResyncXmlElementFactory(IXmlTagContainer xmlTagContainer, IXmlElementFactory factory = null);

  public override IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public override IXmlTag CreateRootTag(IXmlTagHeader header, IXmlElementFactoryContext context);
  public override IXmlTag CreateTag(IXmlTagHeader header, IXmlTag parentTag, IXmlElementFactoryContext context);
  public override IXmlTagFooter CreateTagFooter(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public override IXmlTagHeader CreateTagHeader(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
}
```

<!-- End ResyncXmlElementFactory -->

Implementation detail for incremental reparsing. When a change is made to a file which is encapsulated by an [`IXmlTag`](TreeNodes.md#ixmltag), the tag's `IChameleonNode.Resync` method is called, and the node tries to reparse just its contents, rather than have to parse the whole file on each change.

The changed range is re-parsed, as a whole file. However, the element factory used by the parser is [`ResyncXmlElementFactory`](#resyncxmlelementfactory), which is passed the original parent tag in its constructor. When creating tags, headers, footers and attributes where the passed `parentTag` is the root [`IXmlFile`](TreeNodes.md#ixmlfile), the [`ResyncXmlElementFactory`](#resyncxmlelementfactory) replaces the `parentTag` parameter with the original parent tag passed to the constructor. This keeps the context of the original parent tag when parsing the changed child content.

### ResyncXmlElementFactoryContext

<!-- Begin ResyncXmlElementFactoryContext -->

```cs
public class ResyncXmlElementFactoryContext :
  IXmlElementFactoryContext
{
  public ResyncXmlElementFactoryContext(ITreeNode physicalNode);

  public T GetContainingNode<T>(ITreeNode currentNode, bool returnCurrent);
  public IXmlTag GetParentTag(IXmlTag currentTag);
}
```

<!-- End ResyncXmlElementFactoryContext -->

Defers to [`DefaultXmlElementFactoryContext`](#defaultxmlelementfactorycontext). If the returned value is `null`, the original node passed to the constructor is checked.

## X

### XmlElementFactory

<!-- Begin XmlElementFactory -->

```cs
public class XmlElementFactory
{
  protected XmlElementFactory(IPsiModule module, IXmlElementFactory factory, IXmlLanguageSupport support, bool applyFormatter);

  protected bool ApplyFormatter;
  protected IXmlElementFactory Factory;
  protected IPsiModule Module;
  protected IXmlLanguageSupport Support;

  public IXmlAttribute CreateAttributeForTag(IXmlTag contextTag, string attributeText);
  public IXmlFile CreateFile(string xmlText);
  public IXmlAttribute CreateRootAttribute(string attributeText);
  public IXmlTag CreateRootTag(string tagText);
  public IXmlTag CreateTagForTag(IXmlTag contextTag, string tagText, string rootTagText = null);

  public static XmlElementFactory GetInstance(ITreeNode context, bool applyFormatter = true);
}
```

<!-- End XmlElementFactory -->

Methods for creating instances of [`IXmlFile`](TreeNodes.md#ixmlfile), [`IXmlTag`](TreeNodes.md#ixmltag) and [`IXmlAttribute`](#ixmlattribute). It is intended for use by code client. Note that it does not implement the similarly named [`IXmlElementFactory`](#ixmlelementfactory) interface (in fact, internally, it uses this interface). Clients can get an instance of the class via the `GetInstance` static method. Passing in an `ITreeNode` allows for getting the language specific implementations of [`IXmlElementFactory`](#ixmlelementfactory) and `XmlLanguageSupport`.

* `CreateFile` will parse a string containing XML into a complete XML file, represented by [`IXmlFile`](TreeNodes.md#ixmlfile).
* `CreateRootTag` creates an instance of [`IXmlTag`](TreeNodes.md#ixmltag) based on the given XML string. It is a "root" tag in that it treats the given string as a complete XML file, and returns the first tag found in that "file".
* `CreateRootAttribute` creates an instance of [`IXmlAttribute`](TreeNodes.md#ixmlattribute) given an XML string that represents an attribute, e.g. `name="value"`. It is a "root" attribute in that the attribute is added to a "root" tag in a parsed file.
* `CreateTagForTag` creates an instance of [`IXmlTag`](TreeNodes.md#ixmltag), as though it were parsed with a parent tag of `contextTag`. It does this by creating an instance of the [`XmlElementFactoryForCreateTag`](#xmlelementfactoryforcreatetag) element factory, passing in `contextTag` (and the language specific implementation of [`IXmlElementFactory`](#ixmlelementfactory)). When this factory gets called to create a new tag, it substitutes the `contextTag` as the parent, and passes the call to the inner implementation to create the tag. This allows per-language implementations of [`IXmlElementFactory`](#ixmlelementfactory) to have more context when creating a tag - it knows the parent tag and can create an appropriate child tag. For example, the XAML implementation of [`IXmlElementFactory`](#ixmlelementfactory) creates an instance of `StaticResourceElement` (that implements [`IXmlTag`](TreeNodes.md#ixmltag) if `textTag` is `<x:Static>...</x:Static>` and the parent `contextTag` is a resource dictionary. The `rootTagText` parameter is used to create the root tag, if that is required for special processing. Leaving it null will use a sensible default value.
* `CreateAttributeForTag` is similar to `CreateTagForTag`, except it uses the [`XmlElementFactoryForCreateAttribute`](#xmlelementfactoryforcreateattribute) element factory to use the given `contextTag` when creating an attribute.

Once a node has been created, it can be passed to `ModificationUtil` to be added to a new tree.

Generally speaking, callers should use `CreateTagForTag` and `CreateTagForAttribute` when wishing to create a tag or an attribute. Only use the "root" variants when the tag or attribute actually is at the root level.

### XmlElementFactory&lt;TXmlLanguage&gt;

<!-- Begin XmlElementFactory`1 -->

```cs
public class XmlElementFactory<TXmlLanguage> :
  XmlElementFactory
{
  protected XmlElementFactory<TXmlLanguage>(IPsiModule module, bool applyFormatter);

  public static XmlElementFactory<TXmlLanguage> GetInstance(IPsiModule module, bool applyFormatter = true);
}
```

<!-- End XmlElementFactory`1 -->

A derived instance of [`XmlElementFactory`](#xmlelementfactory) which allows creating calling `GetInstance` for a specific language without requiring an instance of `ITreeNode`. Also derived by some XML languages to provide additional functionality for the factory, e.g. `ResxElementFactory` and `XamlElementFactory`.

### XmlElementFactoryForCreateAttribute

<!-- Begin XmlElementFactoryForCreateAttribute -->

```cs
public class XmlElementFactoryForCreateAttribute :
  DelegatingXmlElementFactory
{
  public XmlElementFactoryForCreateAttribute(IXmlElementFactory factory, IXmlTag tag);

  public override IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
}
```

<!-- End XmlElementFactoryForCreateAttribute -->

Instance of [`IXmlElementFactory`](#ixmlelementfactory) that changes the context when creating an attribute to instead of using the passed in `parentTag` as the parent tag of the newly created attribute, uses an instance of `IXmlTag` passed to the constructor. This is used as an implementation detail of [`XmlElementFactory.CreateAttributeForTag()`](#xmlelementfactory) to make sure the new attribute is created with the correct parent tag as context. See [`XmlElementFactory`](#xmlelementfactory) for more details.

### XmlElementFactoryForCreateTag

<!-- Begin XmlElementFactoryForCreateTag -->

```cs
public class XmlElementFactoryForCreateTag :
  DelegatingXmlElementFactory
{
  public XmlElementFactoryForCreateTag(IXmlElementFactory factory, IXmlTag tag);

  public override IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public override IXmlTag CreateTag(IXmlTagHeader header, IXmlTag parentTag, IXmlElementFactoryContext context);
}
```

<!-- End XmlElementFactoryForCreateTag -->

Instance of [`IXmlElementFactory`](#ixmlelementfactory) that changes the context when creating a tag to ignore the passed `parentTag` when creating the tree node, and use the tag passed to the constructor as the parent instead. This class is used as an implementation detail of [`XmlElementFactory.CreateTagForTag()`](#xmlelementfactory) to make sure the new attribute is created with the correct parent tag as context. See [`XmlElementFactory`](#xmlelementfactory) for more details.

### XmlTreeNodeFactory

<!-- Begin XmlTreeNodeFactory -->

```cs
[Language(typeof(XmlLanguage))]
public class XmlTreeNodeFactory :
  IXmlElementFactory
{
  public XmlTreeNodeFactory(XmlLanguage languageType, XmlTokenTypes tokenTypes, XmlElementTypes elementTypes);

  PsiLanguageType LanguageType { get; set; }
  XmlElementTypes XmlElementType { get; set; }
  XmlTokenTypes XmlTokenTypes { get; set; }

  public IAnyContent CreateAnyContent();
  public IAttDef CreateAttDef();
  public IXmlAttribute CreateAttribute(IXmlIdentifier nameIdentifier, IXmlAttributeContainer attributeContainer, IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlAttributeValue CreateAttributeValue(XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  public IAttType CreateAttType();
  public IXmlCData CreateCData();
  public IXmlComment CreateComment();
  public IDocTypeDeclaration CreateDocTypeDeclaration();
  public IDTDAttListDecl CreateDTDAttListDecl();
  public IDTDElementDecl CreateDTDElementDecl();
  public IDTDEntityDecl CreateDTDEntityDecl();
  public IDTDNotationDecl CreateDTDNotationDecl();
  public IEmptyContent CreateEmptyContent();
  public IXmlSyntaxErrorElement CreateError(XmlSyntaxErrorType errorType);
  public IXmlFile CreateFile();
  public IGrouppedContent CreateGrouppedContent();
  public IXmlIdentifier CreateIdentifier(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);
  public IDTDBody CreateIntSubset();
  public IDTDNDataDecl CreateNDataDecl();
  public IProcessingInstruction CreatePI();
  public IXmlProcessingInstruction CreatePIXml();
  public IExternalId CreatePublicExternalID();
  public IRepetitionContent CreateRepetionContent();
  public IXmlTag CreateRootTag(IXmlTagHeader header, IXmlElementFactoryContext context);
  public IElementContent CreateSimpleContent();
  public IExternalId CreateSystemExternalID();
  public IXmlTag CreateTag(IXmlTagHeader header, IXmlTag parentTag, IXmlElementFactoryContext context);
  public IXmlTagFooter CreateTagFooter(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlTagHeader CreateTagHeader(IXmlTagContainer parentTag, IXmlElementFactoryContext context);
  public IXmlToken CreateToken(ITokenIntern internalizer, XmlTokenNodeType tokenType, IBuffer buffer, int startOffset, int endOffset);

  public static IXmlElementFactory GetInstance<TLanguage>();
  public static IXmlElementFactory GetInstance(ITreeNode element);
}
```

<!-- End XmlTreeNodeFactory -->

Default implementation of [`IXmlElementFactory`](#ixmlelementfactory). This is used by the parser and lexer to create [`IXmlTreeNode`](TreeNode.md#ixmltreenode) instances when building the PSI tree. It is a per-language class, as indicated by the `[Language(typeof(XmlLanguage))]` attribute. This means it will be used for all XML derived languages when calling `LanguageManager.Instance.GetService<IXmlElementFactory>(language)`, unless the derived language implements its own [`IXmlElementFactory`](#ixmlelementfactory).

Call one of the `GetInstance` methods to get an instance of the class (internally, this calls `LanguageManager.GetService`, using the language from the passed in `ITreeNode` or the `TLanguage` generic parameter.

Use this class to create low-level tree nodes. Use the [`XmlElementFactory`] class to create a whole [`IXmlFile`](TreeNodes.md#ixmlfile) or tags and attributes.
