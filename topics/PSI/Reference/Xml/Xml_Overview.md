[//]: # (title: Overview)



## Tree structure

Thanks to the simple and regular structure of XML, the XML PSI tree is fairly small, and pretty easy to understand. However, the PSI tree should not be confused with a DOM. The PSI tree is an abstract syntax tree, and models all of the syntax in an XML file, while an XML DOM is a (light) abstraction on top of the file. A DOM lists elements and attributes, while an abstract syntax tree will also list whitespace, and the constructs used to create an element (e.g. the opening and closing tags).

A good example here is that the text contents of an XML element is split into text tokens and whitespace nodes. Even if the owning XML element has the `xml:space="preserve"` attribute set, the tree doesn't change, and the whitespace nodes are still added. In other words, the semantic nature of the `xml:space` attribute isn't reflected in the abstract syntax tree.

Since XML is a hierarchical structure, care should also be taken not to confuse child nodes in the tree with children in the XML DOM. For example, the children of an XML element in the PSI tree include the nodes used to construct the opening and closing tags, as well as the inner nodes of the elements (whitespace, child tags, etc).

See the [Tree Nodes reference section](TreeNodes.md) for more details of the XML tree nodes.

### Tree nodes

Each node in the XML tree derives from [`IXmlTreeNode`](TreeNodes.md#ixmltreenode):

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

The `XmlTokenTypes` property provides access to the token node type instances assigned to the token nodes when building the tree. These instances are the values returned from `ITreeNode.NodeType`, and `ITokenNode.GetTokenType()`. It can be used to compare against the token node type reported by a token node in order to find a particular token type (e.g. look for tokens of type `XmlTokenTypes.COMMENT_START`). It is also used by the XML file parser to create token nodes via the token node type instances' `Create` method. This might return a derived instance for some XML languages, such as XAML.

The `AcceptVisitor` method implements the [Visitor pattern](http://en.wikipedia.org/wiki/Visitor_pattern). It will call the strongly typed `Visit` method of the `IXmlTreeVisitor` instance passed in.

### Tokens

All tokens in the tree derive from [`IXmlToken`](TreeNodes.md#ixmltoken), which in turn derives from [`IXmlTreeNode`](TreeNodes.md#ixmltreenode). Remember that a token is a leaf element in the tree - it doesn't have any children, and is used to build the higher level tree nodes (e.g. an xml tag contains the tokens for `<`, the tag name identifier, `>` and so on).

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

The `ITokenNode.GetTokenType` method is hidden by a new method of the same name, but that returns an instance of `XmlTokenNodeType`, rather than `TokenNodeType`. `XmlTokenNodeType` derives from the `TokenNodeType` abstract class and adds a single property, that returns the `XmlTokenTypes` class that lists all the known token node types. This class can be a derived instance for some XML languages, such as XAML.

```csharp
public XmlTokenTypes XmlTokenTypes { get; }
```

### File node

The root node of an XML file is [`IXmlFile`](TreeNodes.md#ixmlfile):

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

The `XmlElementTypes` property provides access to the composite node type instances assigned to the composite, non-leaf nodes when building the tree. These instances are the values returned from `ITreeNode.NodeType`. It can be used to compare against the node type of any node, to look for nodes of a certain type, and is also used by the XML file parser to create element nodes via the element node type instances' `Create` method. The type of `XmlElementTypes` might be a derived instance, depending on the XML language.

The `IXmlFile` directly exposes a collection of processing instructions. This only returns processing instructions that implement [`IProcessingInstruction`](TreeNodes.md#iprocessinginstruction), and despite its name, the [`IXmlProcessingInstruction`](TreeNodes.md#ixmlprocessinginstruction) that represents the XML declaration doesn't get returned in this collection. To get at the XML declaration, you need to get it directly from the child nodes:

```csharp
var xmlDeclarations = file.Children<IXmlProcessingInstruction>();
```

The inherited [`IXmlTagContainer`](TreeNodes.md#ixmltagcontainer) interface provides child tag access and manipulation functions.

It also implements the [`IXmlDocumentNode`](TreeNodes.md#ixmldocumentnode) marker interface.

### XML tags

An XML tag is represented with the [`IXmlTag`](TreeNodes.md#ixmltag) interface. This is where it is important not to confuse child tags with child nodes. The [`IXmlTag`](TreeNodes.md#ixmltag) interface represents the span from the opening `<` character to the closing `>` character of an XML element. Its child nodes in the tree split this span further, into a header, an optional footer and the content nodes.

The header is represented with the [`IXmlTagHeader`](TreeNodes.md#ixmltagheader) interface, and covers the span from the opening `<` character to the closing `>` of the **opening** part of the XML element. E.g. `<foo baz="quux">` only, and not including the closing element `</foo>`. This means that the header span also includes attributes, and the tree nodes representing the attributes are indeed child nodes of the tag header.

The footer is the closing element (`</foo>`), which may not be included in the tree if the XML element is self-closed (`<foo />`).

The inner child nodes are either text ([`IXmlFloatingTextTokenNode`](TreeNodes.md#ixmlfloatingtexttokennode), which also includes whitespace), or other nodes such as [`IXmlCData`](TreeNodes.md#ixmlcdata), [`IXmlEntityTokenNode`](TreeNodes.md#ixmlentitytokennode) or [`IXmlTag`](TreeNodes.md#ixmltag).

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

### XML attributes

Attributes are accessible via the [`IXmlAttributeContainer`](TreeNodes.md#ixmlattributecontainer) interface, mostly implemented by [`IXmlTagHeader`](TreeNodes.md#ixmltagheader), but also by the XML declaration [`IXmlProcessingInstruction`](TreeNodes.md#ixmlprocessinginstruction).

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

Methods to add and remove attributes are available on [`IXmlTag`](TreeNodes.md#ixmltag).

### Processing Instructions

Processing instructions are represented with the [`IProcessingInstruction`](TreeNodes.md#iprocessinginstruction) interface. This is a simple interface, providing access to the target name, and a string representing the unparsed content.

## DTD

> **TODO** Write about DTD, inline and standalone

## Derived XML languages

> **TODO** Write about derived XML languages. How does this relate to the other PSI reference sections?

## Manipulating the tree

> **TODO** Write about manipulating the tree. Push this to own page? E.g. element factories

## Navigation

The XML PSI implementation includes several [navigator classes](Navigators.md). As usual, the naming follows a convention - the classes are named after the tree node you are trying to **navigate to**, to find, suffixed with 'Navigator', for example, `XmlTagNavigator`. And the methods are named after the tree node you are **navigating from**, e.g. `XmlTagNavigator.GetByTagHeader`.

## Utilities

The XML PSI implementation includes a couple of [utility classes](Utils.md), some for manipulation of the tree, and some for helper methods for handling references within XML files to CLR types (e.g. web.config files)