# Manipulating the Tree

The XML PSI tree doesn't provide many strongly typed methods for manipulating the tree. The [`IXmlTag`](TreeNodes.md#ixmltag) and [`IXmlTagContainer`](TreeNodes.md#ixmltagcontainer) nodes are the only ones that provide methods.

```cs
public inteface IXmlTag
{
  // ...snip...

  TXmlAttribute AddAttributeAfter<TXmlAttribute>(TXmlAttribute attribute, IXmlAttribute anchor);
  TXmlAttribute AddAttributeBefore<TXmlAttribute>(TXmlAttribute attribute, IXmlAttribute anchor);
  void RemoveAttribute(IXmlAttribute attribute);
}

public interface IXmlTagContainer
{
  // ...snip...
  TXmlTag AddTagAfter<TXmlTag>(TXmlTag, IXmlTag anchor);
  TXmlTag AddTagBefore<TXmlTag>(TXmlTag, IXmlTag anchor);
}
```

Both interfaces allow adding an attribute or a tag, specifying an `anchor` tree node that should be used to insert before or after. This `anchor` node can be null, in which case, the `*After` methods will add the new attribute or tag **before** all other nodes, and the `*Before` methods will add it **after** the other nodes.

> **Info** This may seem counter-intuitive, but if you want to use the `*After` methods to add to the end of the current list, you can pass in the node at the end of the list. Without this `null` handling, there is no way to add an attribute to the start of the list using the `*After` methods, and you would have to introduce special case handling.

## Util classes

While the tree nodes don't provide any more methods for manipulating the tree, the [Util classes](Utils.md) do. Specifically, [`XmlAttributeUtil`](Utils.md#xmlattributeutil) and [`XmlTagUtil`](Utils.md#xmltagutil).

```cs
public static class XmlAttributeUtil
{
  // ...snip...
  public static void SetValue(IXmlAttribute attribute, string unquotedValue);
}

public static class XmlTagUtil
{
  // ...snip...
  public static void MakeCompound(IXmlTag tag);
  public static void MakeEmptyTag(IXmlTag tag);
}
```

The [`XmlAttributeUtil.SetValue`](Utils.md#xmlattributeutil) method sets the value of the attribute to the string passed in. Since there are no methods for manipulating the text of the attribute value, it does this by replacing the [`IXmlAttributeValue`](TreeNodes.md#ixmlattributevalue) node in the given attribute with a new instance.

The [`XmlTagUtil`](Utils.md#xmltagutil) methods rewrite the tree, replacing a tag with an opening and closing element to one which is self-closing, and vice versa.

## Removing nodes

While `IXmlTag.RemoveAttribute` allows for removing an attribute, there isn't a method for removing a child tag, or other nodes, such as processing instructions. In this case, `ModificationUtil.DeleteChild` can be used to remove a specific node, or `ModificationUtil.DeleteChildRange` to remove a range of nodes.

> **Warning** When calling `ModificationUtil` methods, the code expects the caller to have already taken the write lock, if the node being operated on is part of a file. I.e. `node.IsPhysical()` returns `true`. The write lock is not needed if the node isn't part of a file, such as when it has been just created and not yet inserted into the tree, or it has just been removed from the tree.
>
> The `ModificationUtil` methods contain an assert that the write lock has been correctly taken, but the assert is disabled in the production build of ReSharper. It is **highly recommended** to install the checked version when developing extensions, so the assert will run, and verify your code.

## Adding nodes

Adding a new node is fairly straightforward, in that it's a simple call to one of the `ModificationUtil.AddChild*` methods.

> **Warning** Again, ensure that the write lock is held if calling `ModificationUtil.AddChild*` on a node that is already part of a file's tree.

Before adding a node, however, it must be created first. The XML PSI has a couple of methods for creating nodes. The `XmlElementFactory` class can create an instance of `IXmlFile`, and has methods for creating tags and attributes. It does not provide methods for creating other tree node types. You can use the `XmlElementFactory.GetInstance` method to get an instance of the element factory, passing in an existing `ITreeNode` for context.

The `IXmlElementFactory` interface provides more fine-grained methods for creating XML tree nodes (note that `XmlElementFactory` **does not** implement `IXmlElementFactory`). This interface is implemented per-language, for each XML-derived language (such as web.config, build scripts, etc). The default implementation is `XmlTreeNodeFactory`. The `XmlTreeNodeFactory.GetInstance` static methods will get the appropriate language specific implementation.

> **Note** There is also the `XmlElementFactory<TXmlLanguage>` class, which derives from `XmlElementFactory`, and also provides a `GetInstance` method. This method allows for creating an instance of `XmlElementFactory` without using an existing `ITreeNode` to provide the context for which XML-derived language is being used (e.g. web.config, .resx, etc). There are also the `ResxElementFactory` and `XamlElementFactory` which derive from `XmlElementFactory<TXmlLanguage>` and provide extra functionality for `.resx` and XAML files.

See the documentation on [element factories](ElementFactories.md) for more details.

## Modifying nodes

Unless a tree node or utility class provides a helper method for manipulating the tree node, then any time the content of the XML tree needs to be updated, then the node containing that content must be replaced. That is, a new node is created, and replaces the old node in the tree.

This can be quite complex, depending on what exactly needs to be replaced. The `XmlElementFactory` class provides methods to easily create tags and attributes, but not for creating other node types. The language specific implementations of `IXmlElementFactory` can create all of the other node types, but usually requires more work to build the tree nodes required.

For example, replacing the inner text of a tag is complicated by the text being represented by both text token nodes and whitespace token nodes. Replacing a single word is therefore straightforward:

```cs
public void ReplaceTextNode(IXmlTag tag, IXmlFloatingTextTokenNode oldToken, string text)
{
  // Will only take the write lock if tag is part of a "real" tree
  using(WriteLockCookie.Create(tag.IsPhysical()))
  {
    IXmlElementFactory elementFactory = XmlTreeNodeFactory.GetInstance(tag);
    var buffer = new StringBuffer(text);

    // We're not doing enough parsing to make it useful to intern strings more efficiently
    var tokenIntern = new LexerTokenIntern();
    var tokenNodeType = tag.XmlTokenTypes.TEXT;
    var newToken = elementFactory.CreateToken(tokenIntern, tokenNodeType, buffer, 0, buffer.Length);
    ModificationUtil.ReplaceChild(oldToken, newToken);
  }
}
```

However, replacing content that spans whitespace nodes is more involved. A better approach here can be to create a new tag with the entire expected text content, and replace the child nodes of the original tag with the new tag's children. Alternatively, replace the whole tag with the newly created one (ensuring attributes are correctly copied across).

```cs
public void ReplaceTextContents(IXmlTag tag, string text)
{
  using(WriteLockCookie.Create(tag.IsPhysical()))
  {
    // Get an instance of XmlElementFactory, NOT IXmlElementFactory
    XmlElementFactory elementFactory = XmlElementFactory.GetInstance(tag);

    // Create the text to be used to create the tag
    var tagText = string.Format("<foo>{0}</foo>", text);
    var newTag = elementFactory.CreateRootTag(tagText);

    var oldTextTokens = tag.InnerTextTokens;
    ModificationUtil.DeleteChildRange(oldTextTokens.First(), oldTextTokens.Last());

    var newTextTokens = newTag.InnerTextTokens;
    ModificationUtil.AddChildRangeAfter(tag.Header, newTextTokens);
  }
}
```

