# ElementFactories


<!-- toc -->
<!-- toc stop -->

## C

### ClrDocCommentElementFactoryImpl

<!-- Begin ClrDocCommentElementFactoryImpl -->

```cs
public class ClrDocCommentElementFactoryImpl :
  DocCommentElementFactory
{
  protected new ClrDocCommentElementFactoryImpl(IDocCommentXmlPsi xmlPsi);

  Key<Object> XmlResolveKey { get; }

  public IXmlTag CreateException(IDeclaredType type);

  public IList<DecodeInfo> DecodeCRefs(IXmlTagContainer file);
}
```

<!-- End ClrDocCommentElementFactoryImpl -->

## D

### DocCommentElementFactory

<!-- Begin DocCommentElementFactory -->

```cs
public class DocCommentElementFactory
{
  protected DocCommentElementFactory(IDocCommentXmlPsi xmlPsi);

  protected XmlElementFactory Factory;

  public IXmlTag CreateParameterNode(string name, string innerText = null);
  public IXmlTag CreateReturnNode(string text = null);
  public IXmlTag CreateTag(string text);
  public IXmlTag CreateTypeParameterNode(string name, string innerText = null);
  public IXmlTag CreateValueNode(string text = null);

  public static DocCommentElementFactory GetInstance(IDocCommentXmlPsi docCommentXmlPsi);
}
```

<!-- End DocCommentElementFactory -->

