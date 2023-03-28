[//]: # (title: Tree Nodes \(XML\)))

<!-- Not auto-indexed, so we can split definitions between XML and XML DTD.
     Also not including IDocCommentBlockWithPsi`2 as this is XML Doc Comment PSI stuff -->



## P

### IProcessingInstruction

<!-- Begin IProcessingInstruction -->

```csharp
public interface IProcessingInstruction :
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken Body { get; }
  IXmlToken End { get; }
  string InstructionTarget { get; }
  IXmlIdentifier InstructionTargetNode { get; }
  IXmlToken Start { get; }
}
```

* See also: [IXmlIdentifier](TreeNodes.md#ixmlidentifier), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IProcessingInstruction -->

Represents a [processing instruction](http://en.wikipedia.org/wiki/Processing_Instruction). The `Start` and `End` properties give access to the tokens that represent the start and end of the processing instruction, namely the `<?` and `?>` tokens respectively. The `InstructionTargetNode` property is an [`IXmlIdentifier`](#ixmlidentifier) that represents the target of the processing instruction, that is, the identifier immediately following the start token. As a convenience, you can get at the text of the target using the `InstructionTarget` property, which simply calls `GetText()` on the [`IXmlIdentifier`](#ixmlidentifier). Finally, the `Body` property is a token that contains the content of the processing instruction, if any. This is a simple `XmlToken` containing the text after the target, unparsed. If the instruction contains text that looks like attributes, they are not parsed - the `Body` property will contain the full text.

While the `<?xml ... ?>` declaration at the start of an XML document shares similar syntax to processing instructions, it is not a processing instruction, and is instead represented using the [`IXmlProcessingInstruction`](#ixmlprocessinginstruction) interface.

## X

### IXmlAsteriskTokenNode

<!-- Begin IXmlAsteriskTokenNode -->

```csharp
public interface IXmlAsteriskTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlAsteriskTokenNode -->

Marker interface for a token representing the asterisk character (`*`).

### IXmlAttribute

<!-- Begin IXmlAttribute -->

```csharp
public interface IXmlAttribute :
  IXmlTreeNode,
  ITreeNode
{
  string AttributeName { get; }
  IXmlToken Eq { get; }
  IXmlIdentifier Identifier { get; }
  string UnquotedValue { get; }
  IXmlAttributeValue Value { get; }
  string XmlName { get; }
  TreeTextRange XmlNameRange { get; }
  string XmlNamespace { get; }
  TreeTextRange XmlNamespaceRange { get; }
}
```

* See also: [IXmlAttributeValue](TreeNodes.md#ixmlattributevalue), [IXmlIdentifier](TreeNodes.md#ixmlidentifier), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlAttribute -->

Represents an attribute. The `Identifier` property returns the tree node that contains the name of the attribute, and the `AttributeName` convenience property will retrieve the full text of the attribute name, including namespace prefix, if applicable.

The `XmlName` and `XmlNamespace` properties will retrieve the text of the attribute name split into local name and namespace prefix, respectively. E.g. `foo:bar="quux"` will return `"bar"` for `XmlName` and `"foo"` for `XmlNamespace`. If there is no namespace prefix, an empty string is returned (the `XmlNamespace` property is annotated with `[NotNull]`).

The `XmlNameRange` and `XmlNamespaceRange` properties return the ranges of the `XmlName` and `XmlNamespace` strings relative to the `Identifier` node. That is, they are the offsets into `AttributeName`.

The `Value` property will return the [`IXmlAttributeValue`](#ixmlattributevalue) node representing the value, while the `UnquotedValue` property will return the string value of the attribute, without surround quotes.

Finally, the `Eq` property is the token representing the equals sign in the attribute declaration, e.g. `name="value"`.

### IXmlAttributeContainer

<!-- Begin IXmlAttributeContainer -->

```csharp
public interface IXmlAttributeContainer :
  IXmlTreeNode,
  ITreeNode
{
  TreeNodeCollection<IXmlAttribute> Attributes { get; }
  TreeNodeEnumerable<IXmlAttribute> AttributesEnumerable { get; }
  string ContainerName { get; }
}
```

* See also: [IXmlAttribute](TreeNodes.md#ixmlattribute), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlAttributeContainer -->

Represents a node that can contain attributes. This is typically mostly inherited by [`IXmlTagHeader`](#ixmltagheader), but also by [`IXmlProcessingInstruction`](#ixmlprocessinginstruction) in order to expose the attributes defined in the XML declaration. The `Attributes` collection is pre-iterated and backed by an array, while the `AttributesEnumerable` is lazily evaluated.

The `ContainerName` property returns the text of the child [`IXmlIdentifier`](#ixmlidentifier) node of the container. In other words, for an XML tag, it will return the name of the tag. For an XML declaration, it will return the "xml" part of the `<?xml ... ?>` declaration.

### IXmlAttributeValue

<!-- Begin IXmlAttributeValue -->

```csharp
public interface IXmlAttributeValue :
  IXmlTreeNode,
  ITreeNode
{
  string UnquotedValue { get; }
  IXmlValueToken ValueToken { get; }
}
```

* See also: [IXmlTreeNode](TreeNodes.md#ixmltreenode), [IXmlValueToken](TreeNodes.md#ixmlvaluetoken)

<!-- End IXmlAttributeValue -->

Represents the value in an attribute. It is implemented by the `XmlValueToken` type, and represents the quoted part of the value in an XML attribute `name="value"`. The `ValueToken` property provides access to an instance of [`IXmlValueToken`](#ixmlvaluetoken) token that essentially represents the same information (indeed, the `XmlValueToken` implementation simply returns `this`).

### IXmlCData

<!-- Begin IXmlCData -->

```csharp
public interface IXmlCData :
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken Body { get; }
  string CData { get; }
  IXmlToken End { get; }
  IXmlToken Start { get; }
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCData -->

An XML [CDATA](http://en.wikipedia.org/wiki/CDATA) node. The `Start` property is the `<![CDATA[` token, and `End` is the `]]>` token. While they are listed as [`IXmlToken`]() instances, they are in fact, instances of the [`IXmlCdataStartTokenNode`](#ixmlcdatastarttokennode) and [`IXmlCdataEndTokenNode`](#ixmlcdataendtokennode) marker interfaces. `Body` is a token representing the content of the CDATA node, while the `CData` property is the content as a plain string.

### IXmlCdataEndTokenNode

<!-- Begin IXmlCdataEndTokenNode -->

```csharp
public interface IXmlCdataEndTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCdataEndTokenNode -->

Marker interface for the CDATA end token `]]>`.

### IXmlCdataStartTokenNode

<!-- Begin IXmlCdataStartTokenNode -->

```csharp
public interface IXmlCdataStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCdataStartTokenNode -->

Marker interface for the CDATA start token `<![CDATA[`.

### IXmlCommaTokenNode

<!-- Begin IXmlCommaTokenNode -->

```csharp
public interface IXmlCommaTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCommaTokenNode -->

Marker interface for a token representing the comma character (`,`).

### IXmlComment

<!-- Begin IXmlComment -->

```csharp
public interface IXmlComment :
  IXmlCommentNode,
  IXmlTreeNode,
  ITreeNode
{
}
```

* See also: [IXmlCommentNode](TreeNodes.md#ixmlcommentnode), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlComment -->

Represents an XML comment. See [`IXmlCommentNode`](#ixmlcommentnode).

### IXmlCommentEndTokenNode

<!-- Begin IXmlCommentEndTokenNode -->

```csharp
public interface IXmlCommentEndTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCommentEndTokenNode -->

Marker interface for the XML comment close token `-->`.

### IXmlCommentNode

<!-- Begin IXmlCommentNode -->

```csharp
public interface IXmlCommentNode :
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken CommentBody { get; }
  IXmlToken CommentEnd { get; }
  IXmlToken CommentStart { get; }
  string CommentText { get; }
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCommentNode -->

Represents and XML comment. The `CommentStart` and `CommentEnd` properties are the start (`<!--`) and end (`-->`) tokens of the comment. Although listed here as [`IXmlToken`](#ixmltoken), they are in fact instances of the [`IXmlCommentStartTokenNode`](#ixmlcommentstarttokennode) and [`IXmlCommentEndTokenNode`](#ixmlcommentendtokennode) marker interfaces. The comment text is held in the `CommentBody` node, and exposed as a string with the `CommentText` property.

### IXmlCommentStartTokenNode

<!-- Begin IXmlCommentStartTokenNode -->

```csharp
public interface IXmlCommentStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlCommentStartTokenNode -->

Marker interface for the XML comment start token `<!--`.

### IXmlDocumentNode

<!-- Begin IXmlDocumentNode -->

```csharp
public interface IXmlDocumentNode :
  IXmlTreeNode,
  ITreeNode
{
}
```

* See also: [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlDocumentNode -->

Marker interface representing the document node of an XML file. Inherited by [`IXmlFile`](#ixmlfile).

### IXmlDtdStartTokenNode

<!-- Begin IXmlDtdStartTokenNode -->

```csharp
public interface IXmlDtdStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlDtdStartTokenNode -->

Marker interface for the start token (`<!DOCTYPE`) of an inline DTD declaration.

### IXmlEntityTokenNode

<!-- Begin IXmlEntityTokenNode -->

```csharp
public interface IXmlEntityTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
  bool IsNumericEntity { get; }
  string Name { get; }
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlEntityTokenNode -->

An XML [entity reference token](http://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references) representing content such as `&#8226;` or `&copy;`. The `IsNumericEntity` property is true if the entity begins with the string `&#`. The `Name` property is the name of the entity, with the ampersand and semi-colon removed, or `null` for numeric entities. That is, `&copy;` will return `"copy"` and `&#8226;` will return `null` for `Name`.

### IXmlEqTokenNode

<!-- Begin IXmlEqTokenNode -->

```csharp
public interface IXmlEqTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlEqTokenNode -->

Marker interface for a token representing the equals character (`=`).

### IXmlFile

<!-- Begin IXmlFile -->

```csharp
public interface IXmlFile :
  IFile,
  IXmlTagContainer,
  IXmlDocumentNode,
  IXmlTreeNode,
  ITreeNode
{
  TreeNodeCollection<IProcessingInstruction> ProcessingInstructions { get; }
  XmlElementTypes XmlElementTypes { get; }
}
```

* See also: [IProcessingInstruction](TreeNodes.md#iprocessinginstruction), [IXmlDocumentNode](TreeNodes.md#ixmldocumentnode), [IXmlTagContainer](TreeNodes.md#ixmltagcontainer), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlFile -->

The root interface for an XML file. Derives from both `IFile` and [`IXmlTreeNode`](#ixmltreenode). Also derives from [`IXmlTagContainer`](#ixmltagcontainer) showing that it can have child tags.

Exposes a collection of processing instructions and the collection of element types used by this flavour of XML. See the [overview](Xml_Overview.md) for more information about the element types.

### IXmlFloatingTextTokenNode

<!-- Begin IXmlFloatingTextTokenNode -->

```csharp
public interface IXmlFloatingTextTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlFloatingTextTokenNode -->

Marker interface for a non-fixed length token, e.g. identifiers, text, entity references or whitespace. Note that the default implementation (`XmlFloatingTextToken`) is used for text, while the derived `XmlWhitespaceToken`, `XmlEntityToken` and `XmlIdentifier` are used for the other node types. This is so that they can implement other interfaces, such as [`IXmlEntityTokenNode`](#ixmlentitytokennode) and [`IXmlIdentifier`](#ixmlidentifier). Or in the case of `XmlWhitespaceToken`, to override `ITreeNode.IsFiltered()` to return `true`.

### IXmlIdentifier

<!-- Begin IXmlIdentifier -->

```csharp
public interface IXmlIdentifier :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
  string XmlName { get; }
  TreeTextRange XmlNameRange { get; }
  string XmlNamespace { get; }
  TreeTextRange XmlNamespaceRange { get; }
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlIdentifier -->

An XML identifier, with optional namespace prefix. The `XmlName` property returns the text of the identifier without the namespace prefix, while the `XmlNamespace` property returns the prefix, if any. E.g. `"foo:bar"` will return `"bar"` for `XmlName` and `"foo"` for `XmlNamespace`. If there is no namespace prefix, `XmlNamespace` returns an empty string (the property is marked with the `[NotNull]` annotation attribute).

The `XmlNameRange` and `XmlNamespaceRange` properties return the ranges of the namespace and local name, relative to the start of the identifier node. If there is no namespace prefix, the range returned by `XmlNamespaceRange` is empty.

### IXmlLbracketTokenNode

<!-- Begin IXmlLbracketTokenNode -->

```csharp
public interface IXmlLbracketTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlLbracketTokenNode -->

Marker interface for a token representing the left bracket character (`[`).

### IXmlLparenthTokenNode

<!-- Begin IXmlLparenthTokenNode -->

```csharp
public interface IXmlLparenthTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlLparenthTokenNode -->

Marker interface for a token representing the left parenthesis character (`(`).

### IXmlOrTokenNode

<!-- Begin IXmlOrTokenNode -->

```csharp
public interface IXmlOrTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlOrTokenNode -->

Marker interface for a token representing the "or" or "pipe" character (`|`).

### IXmlPercentTokenNode

<!-- Begin IXmlPercentTokenNode -->

```csharp
public interface IXmlPercentTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlPercentTokenNode -->

Marker interface for a token representing the percent character (`%`).

### IXmlPiendTokenNode

<!-- Begin IXmlPiendTokenNode -->

```csharp
public interface IXmlPiendTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlPiendTokenNode -->

Marker interface for the processing instruction end token `?>`.

### IXmlPistartTokenNode

<!-- Begin IXmlPistartTokenNode -->

```csharp
public interface IXmlPistartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlPistartTokenNode -->

Marker interface for the processing instruction start token `<?`.

### IXmlPlusTokenNode

<!-- Begin IXmlPlusTokenNode -->

```csharp
public interface IXmlPlusTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlPlusTokenNode -->

Marker interface for a token representing the plus character (`+`).

### IXmlProcessingInstruction

<!-- Begin IXmlProcessingInstruction -->

```csharp
public interface IXmlProcessingInstruction :
  IXmlAttributeContainer,
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken EndNode { get; }
  IXmlIdentifier Name { get; }
  IXmlToken StartNode { get; }
}
```

* See also: [IXmlAttributeContainer](TreeNodes.md#ixmlattributecontainer), [IXmlIdentifier](TreeNodes.md#ixmlidentifier), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlProcessingInstruction -->

Represents the XML declaration at the start of an XML file, usually of the form:

```xml
<?xml version="1.0" encoding="utf-8" ?>
```

The `StartNode` and `EndNode` properties provide the [`IXmlPistartTokenNode`](#ixmlpistarttokennode) and [`IXmlPiendTokenNode`](#ixmlpiendtokennode) tokens. The `Name` property gets the [`IXmlIdentifier`](#ixmlidentifier) node that represents the `xml` in the declaration.

Access to the attributes is available via the derived [`IXmlAttributeContainer`](#ixmlattributecontainer) interface members.

### IXmlQuestionTokenNode

<!-- Begin IXmlQuestionTokenNode -->

```csharp
public interface IXmlQuestionTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlQuestionTokenNode -->

Marker interface for a token representing the question mark character (`?`).

### IXmlRbracketTokenNode

<!-- Begin IXmlRbracketTokenNode -->

```csharp
public interface IXmlRbracketTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlRbracketTokenNode -->

Marker interface for a token representing the right bracket character (`]`).

### IXmlRparenthTokenNode

<!-- Begin IXmlRparenthTokenNode -->

```csharp
public interface IXmlRparenthTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlRparenthTokenNode -->

Marker interface for a token representing the right parenthesis character (`)`).

<!--` (keeps vim's syntax highlighting happy after the chars above -->

### IXmlSyntaxErrorElement

<!-- Begin IXmlSyntaxErrorElement -->

```csharp
public interface IXmlSyntaxErrorElement :
  ITreeNode
{
  string ErrorDescription { get; }
  XmlSyntaxErrorType ErrorType { get; }
}
```

<!-- End IXmlSyntaxErrorElement -->

Represents a syntax error in the tree. Does not derive from [`IXmlTreeNode`](#ixmltreenode). The `ErrorDescription` property provides textual representation of the error, which can be displayed to the user, e.g. as a tooltip. The `ErrorType` property is an instance of `XmlSyntaxErrorType`, which can be compared against the static field instances available on the `XmlSyntaxErrorType` class.

### IXmlTag

<!-- Begin IXmlTag -->

```csharp
public interface IXmlTag :
  IXmlTreeNode,
  IXmlTagContainer,
  ITreeNode
{
  IXmlTagFooter Footer { get; }
  IXmlTagHeader Header { get; }
  string InnerText { get; }
  TreeNodeCollection<IXmlToken> InnerTextTokens { get; }
  string InnerValue { get; }
  ITreeRange InnerXml { get; }
  bool IsEmptyTag { get; }

  TXmlAttribute AddAttributeAfter<TXmlAttribute>(TXmlAttribute attribute, IXmlAttribute anchor);
  TXmlAttribute AddAttributeBefore<TXmlAttribute>(TXmlAttribute attribute, IXmlAttribute anchor);

  void RemoveAttribute(IXmlAttribute attribute);
}
```

* See also: [IXmlAttribute](TreeNodes.md#ixmlattribute), [IXmlTagContainer](TreeNodes.md#ixmltagcontainer), [IXmlTagFooter](TreeNodes.md#ixmltagfooter), [IXmlTagHeader](TreeNodes.md#ixmltagheader), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTag -->

Represents a complete XML element. The `Header` and `Footer` properties represent the opening and closing tags of the element. Attributes can be retrieved via the `Header` property (see [`IXmlTagHeader`](#ixmltagheader)).

The inner content can be retrieved in several ways:

* `InnerXml` provides a tree range (relative to the tree) of all nodes after the header, and before the footer, including whitespace. This span will encompass all immediate child nodes (and therefore all descendant nodes, too).
* `InnerText` will return the text from the range returned by `InnerXml`. The text will therefore include all child and descendant tags and content.
* `InnerTextTokens` will return all token nodes (see [`IXmlToken`](#ixmltoken)) that are deemed to be "text" tokens. It does this by calling `IXmlToken.GetTokenType()` and looking that type up in the `tag.XmlTokenTypes.TEXT_NODES` set. This set is initialised to `XmlTokenTypes.TEXT`, `XmlTokenTypes.ENTITY_REF` and `XmlTokenTypes.CHAR_REF`. This means, it will include all instances of [`IXmlFloatingTextTokenNode`](#ixmlfloatingtexttokennode) and [`IXmlEntityTokenNode`](#ixmlentitytokennode).
* `InnerValue` returns a string containing the content of the child text nodes, including text, entity references, whitespace and CDATA nodes. This property will also check to see if whitespace should be preserved, by using the `XmlLangaugeSupport.IsSpacePreserved` method to look for `xml:space="preserved"` on parent tags.

The `IsEmptyTag` property will return true if `Header.IsClosed` returns true. That is, if the tag is of the form `<foo />`.

The `AddAttributeAfter`, `AddAttributeBefore` and `RemoveAttribute` methods will add or remove attributes from the tag header. Passing `null` as the anchor parameter will insert the attribute before the first attribute for `AddAttributeAfter` and after the last attribute for `AddAttributeBefore`. The attribute must have been created beforehand, using the `XmlElementFactory` class. If the tag is part of a physical tree (that is, associated with a file, and not created as part of a sandbox), then the implementations must take the write lock before adding or removing attributes. The modifications can be performed using `ModificationUtil.AddChildAfter`, `ModificationUtil.AddChildBefore` and `ModificationUtil.DeleteRange`.

Child tags can be retrieved and modified using the [`IXmlTagContainer`](#ixmltagcontainer) derived interface.

### IXmlTagContainer

<!-- Begin IXmlTagContainer -->

```csharp
public interface IXmlTagContainer :
  ITreeNode
{
  TreeNodeCollection<IXmlTag> InnerTags { get; }

  TXmlTag AddTagAfter<TXmlTag>(TXmlTag tag, IXmlTag anchor);
  TXmlTag AddTagBefore<TXmlTag>(TXmlTag tag, IXmlTag anchor);

  IList<T> GetNestedTags<T>(string xpath);
  IXmlTag GetTag(Predicate<IXmlTag> predicate);
  TreeNodeEnumerable<T> GetTags<T>();
  TreeNodeCollection<T> GetTags2<T>();

  void RemoveTag(IXmlTag tag);
}
```

* See also: [IXmlTag](TreeNodes.md#ixmltag)

<!-- End IXmlTagContainer -->

Represents a node that can contain tags. Mostly inherited by [`IXmlTag`](#ixmltag), but also inherited by [`IXmlFile`](#ixmlfile).

This interface provides several means of retrieving child tags. Unless otherwise stated, all tags are immediate children of the current tag. When a method is declared as generic, it will filter the child tags so that they match the type passed in. This type must implement [`IXmlTag`](#ixmltag), and is mostly useful for finding a specific tag when working with XML-derived languages, such as web.config files or build scripts.

* The `InnerTags` property provides a (pre-iterated) collection of child [`IXmlTag`](#ixmltag) instances.
* `GetTag(Predicate<IXmlTag> predicate)` will return the first child tag that matches the given predicate.
* `GetTags<T>()` will return a lazily-evaluated enumerable of child tags, filtered to instances of the generic type `T` passed to the method.
* `GetTags2<T>()` will return a pre-iterated collection of child tags. It is similar to `InnerTags`, except it will filter all instances to the type passed in as `T`.
* `GetNestedTags<T>(string xpath)` will return a list of [`IXmlTag`](#ixmltag) instances that match a very simple XPath-like expression. The expression only supports `"*"` and `"/"`, and must start with a tag name. The implementation (`XPathUtil.GetNestedTags<T>()`) will split the path on the `"/"` char, and match each path segment. If the path segment is `"*"` it will match any tag at that level. It does not provide support for matching attributes or skipping path segments with `"//"`. Again, the found tags must also match the type of `T` passed to the method.

The `AddTagAfter`, `AddTagBefore` and `RemoveTag` methods provide means of adding and removing already created tags to the tag container. Implementations will ensure that tag container is not a self-closed tag (using `XmlTagUtil.MakeCompound`) and will then use methods in `XmlContainerUtil` to add or remove the tags. Implementations will also ensure that the write lock is taken, if the tag container is part of a physical tree (that is, associated with a file, and not a sandbox). Tags can be created using the `XmlElementFactory` derived classes.

### IXmlTagEnd1TokenNode

<!-- Begin IXmlTagEnd1TokenNode -->

```csharp
public interface IXmlTagEnd1TokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagEnd1TokenNode -->

Marker interface for a self-closing XML tag end token `/>`.

### IXmlTagEndTokenNode

<!-- Begin IXmlTagEndTokenNode -->

```csharp
public interface IXmlTagEndTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagEndTokenNode -->

Marker interface for an XML tag end token `>`.

### IXmlTagFooter

<!-- Begin IXmlTagFooter -->

```csharp
public interface IXmlTagFooter :
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken EndNode { get; }
  IXmlIdentifier Name { get; }
  IXmlToken StartNode { get; }
}
```

* See also: [IXmlIdentifier](TreeNodes.md#ixmlidentifier), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagFooter -->

Represents the closing tag of an XML element. Not present if the XML tag is self-closing/empty. The `StartNode` is an instance of [`IXmlTagStart1TokenNode`](#ixmltagstart1tokennode) representing the `</` and `EndNode` is the closing `>` [`IXmlTagEndTokenNode`](#ixmltagendtokennode) token.

The `Name` property is the identifier representing the name of the element being closed.

### IXmlTagHeader

<!-- Begin IXmlTagHeader -->

```csharp
public interface IXmlTagHeader :
  IXmlAttributeContainer,
  IXmlTreeNode,
  ITreeNode
{
  IXmlToken EndNode { get; }
  bool IsClosed { get; }
  IXmlIdentifier Name { get; }
  IXmlToken StartNode { get; }
}
```

* See also: [IXmlAttributeContainer](TreeNodes.md#ixmlattributecontainer), [IXmlIdentifier](TreeNodes.md#ixmlidentifier), [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagHeader -->

Represents the opening tag of an XML element. I.e. given the following XML:

```xml
<foo baz="quux">
</foo>
<bar baz="quux" />
```

The text `<foo baz="quux">` will be an instance of `IXmlTagHeader`. The `StartNode` and `EndNode` properties represent the `<` and `>` characters, and will be instances of the [`IXmlTagStartTokenNode`](#ixmltagstarttokennode) and [`IXmlTagEndTokenNode`](#ixmltagendtokennode) marker interfaces.

The text `<bar baz="quux" />` will also be an instance of `IXmlTagHeader`, but will return an instance of [`IXmlTagEnd1TokenNode`](#ixmltagend1tokennode) to represent the closing `/>`. It will also return `true` to the `IsClosed` property.

The `Name` property retrieves the [`IXmlIdentifier`](#ixmlidentifier) token that represents the name of the tag. The text for the name can then be retrieved using `Name.XmlName` and `Name.XmlNamespace` properties.

Attributes are accessible via the inherited [`IXmlAttributeContainer`](#ixmlattributecontainer) interface.

### IXmlTagStart1TokenNode

<!-- Begin IXmlTagStart1TokenNode -->

```csharp
public interface IXmlTagStart1TokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagStart1TokenNode -->

Marker interface for the start token of a closing XML tag (`</`).

### IXmlTagStartTokenNode

<!-- Begin IXmlTagStartTokenNode -->

```csharp
public interface IXmlTagStartTokenNode :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlTagStartTokenNode -->

Marker interface for the start token of an opening XML tag (`</`).

### IXmlToken

<!-- Begin IXmlToken -->

```csharp
public interface IXmlToken :
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
  new XmlTokenNodeType GetTokenType();
}
```

* See also: [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlToken -->

Represents a token in the XML PSI tree. A token is a leaf node in the tree, and cannot be split into smaller nodes. It is the smallest building block node used when creating the tree. For example:

```xml
<foo baz="quux" />
```

When parsed, this is split into the following tokens: `<`, `foo`, `baz`, `=`, `"`, `quux`, `"` and `/>`. These tokens are then used to build higher level constructs in the PSI tree - a tag, with a start token, identifier, attributes (which themselves are composed of tokens) and a close tag token.

The `GetTokenType` method returns the `XmlTokenNodeType` of the particular token. It will be equal to one of the properties set on `XmlTokenTypes`. Note that this method overrides and hides the `ITokenNode.GetTokenType` method, however, the returned `XmlTokenNodeType` derives from `TokenNodeType` and also implements `ITokenNodeType`. One useful method for the returned token type is to check if the node is a whitespace node - `node.GetTokenType().IsWhitespace()`.

### IXmlTreeNode

<!-- Begin IXmlTreeNode -->

```csharp
public interface IXmlTreeNode :
  ITreeNode
{
  XmlTokenTypes XmlTokenTypes { get; }

  TReturn AcceptVisitor<TContext, TReturn>(IXmlTreeVisitor<TContext, TReturn> visitor, TContext context);
}
```

<!-- End IXmlTreeNode -->

Root interface for all XML tree nodes, from tokens to other non-leaf nodes. The `XmlTokenTypes` property retrieves the instance of `XmlTokenTypes` used to construct the tree. This might be a derived instance in the cases that the XML dialect defines new constructs (e.g. XAML).

The `AcceptVisitor` method implements the [Visitor pattern](http://en.wikipedia.org/wiki/Visitor_pattern). It will call the strongly typed `Visit` method of the `IXmlTreeVisitor` instance passed in.

### IXmlValueToken

<!-- Begin IXmlValueToken -->

```csharp
public interface IXmlValueToken :
  IXmlToken,
  IXmlTreeNode,
  ITokenNode,
  ITreeNode
{
  string UnquotedValue { get; }
  TreeTextRange UnquotedValueRange { get; }
}
```

* See also: [IXmlToken](TreeNodes.md#ixmltoken), [IXmlTreeNode](TreeNodes.md#ixmltreenode)

<!-- End IXmlValueToken -->

Represents a token that is a quoted value. Mostly used to represent the value in an attribute. The `UnquotedValue` property provides the content of the token without the surrounding quotes, while the `UnquotedValueRange` property provides the offsets to the unquoted value, relative to the start of the node.
