# Navigators

<!-- Index A - Z (auto-generated. Remove this line if manually adding/removing entries) -->

<!-- toc -->
<!-- toc stop -->

## X

### XmlAttributeNavigator

<!-- Begin XmlAttributeNavigator -->

```cs
public static class XmlAttributeNavigator
{
  public static IXmlAttribute GetByAttributeValue(IXmlAttributeValue attributeValue);
}
```

<!-- End XmlAttributeNavigator -->

Navigate up the tree to an instance of `IXmlAttribute`, given an `IXmlAttributeValue`. Returns `null` if it can't find the `IXmlAttribute`.

### XmlTagContainerNavigator

<!-- Begin XmlTagContainerNavigator -->

```cs
public static class XmlTagContainerNavigator
{
  public static IXmlTagContainer GetByTag(IXmlTag tag);
}
```

<!-- End XmlTagContainerNavigator -->

Navigate up the tree to an instance of `IXmlTagContainerNavigator` given an `IXmlTag`. Returns `null` if it can't find the `IXmlTagContainer`.

### XmlTagNavigator

<!-- Begin XmlTagNavigator -->

```cs
public static class XmlTagNavigator
{
  public static IXmlTag GetByAttribute(IXmlAttribute attribute);
  public static IXmlTag GetByTag(IXmlTag tag);
  public static IXmlTag GetByTagFooter(IXmlTagFooter footer);
  public static IXmlTag GetByTagHeader(IXmlTagHeader header);
}
```

<!-- End XmlTagNavigator -->

Navigate up the tree to an instance of `IXmlTag`, given an instance of `IXmlAttribute`, `IXmlTag`, `IXmlTagFooter` or `IXmlTagHeader`. Returns `null` if it can't find the `IXmlTag`.
