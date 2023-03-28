[//]: # (title: Navigators)

<!-- Index A - Z (auto-generated. Remove this line if manually adding/removing entries) -->



## X

### XmlAttributeNavigator

<!-- Begin XmlAttributeNavigator -->

```csharp
public static class XmlAttributeNavigator
{
  public static IXmlAttribute GetByAttributeValue(IXmlAttributeValue attributeValue);
}
```

* See also: [IXmlAttribute](TreeNodes.md#ixmlattribute), [IXmlAttributeValue](TreeNodes.md#ixmlattributevalue)

<!-- End XmlAttributeNavigator -->

Navigate up the tree to an instance of `IXmlAttribute`, given an `IXmlAttributeValue`. Returns `null` if it can't find the `IXmlAttribute`.

### XmlTagContainerNavigator

<!-- Begin XmlTagContainerNavigator -->

```csharp
public static class XmlTagContainerNavigator
{
  public static IXmlTagContainer GetByTag(IXmlTag tag);
}
```

* See also: [IXmlTag](TreeNodes.md#ixmltag), [IXmlTagContainer](TreeNodes.md#ixmltagcontainer)

<!-- End XmlTagContainerNavigator -->

Navigate up the tree to an instance of `IXmlTagContainerNavigator` given an `IXmlTag`. Returns `null` if it can't find the `IXmlTagContainer`.

### XmlTagNavigator

<!-- Begin XmlTagNavigator -->

```csharp
public static class XmlTagNavigator
{
  public static IXmlTag GetByAttribute(IXmlAttribute attribute);
  public static IXmlTag GetByTag(IXmlTag tag);
  public static IXmlTag GetByTagFooter(IXmlTagFooter footer);
  public static IXmlTag GetByTagHeader(IXmlTagHeader header);
}
```

* See also: [IXmlAttribute](TreeNodes.md#ixmlattribute), [IXmlTag](TreeNodes.md#ixmltag), [IXmlTagFooter](TreeNodes.md#ixmltagfooter), [IXmlTagHeader](TreeNodes.md#ixmltagheader)

<!-- End XmlTagNavigator -->

Navigate up the tree to an instance of `IXmlTag`, given an instance of `IXmlAttribute`, `IXmlTag`, `IXmlTagFooter` or `IXmlTagHeader`. Returns `null` if it can't find the `IXmlTag`.
