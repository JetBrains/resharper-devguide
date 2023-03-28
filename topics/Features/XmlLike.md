[//]: # (title: Working with XML-like files)

ReSharper comes with support for XML and XML-specific notations such as HTML, XAML and Web.config editing. Consequently, you can use the ReSharper API in order to manipulate XML documents.



## Web vs XML

The ReSharper code model distinguishes between two types of documents - web documents (i.e., documents concerning HTML) and XML documents. The distinction is important because, while XML documents are typically concerned with XML only, web documents can include many different notations in addition to HTML. Examples of such notations include JavaScript as well as mark-up for the MVC WebForms or Razor view engines.

Let's take a look at the XML format first.

## XML

Compared to HTML, XML is a lot easier to work with - it has a predictable structure and fairly limited variability in terms of content. Consequently, the API structures for manipulating XML are very straightforward.

Let's take a look at how one would write a context action to support the conversion of an empty tag such as `<test></test>` to a self-closing tag `<test/>`.

The first thing you do is create a class that is marked with the `ContextAction` attribute and is set to inherit from `XmlContextAction`.

```csharp
[ContextAction(
  Group = "XML",
  Name = "Collapse empty tag",
  Description = "Converts from empty opening and closing tags into empty tag",
  Priority = 0)]
public class MakeEmptyTagContextAction : XmlContextAction
{
  public MakeEmptyTagContextAction(XmlContextActionDataProvider dataProvider)
    : base(dataProvider)
  {
  }
}
```

As you can see, the context action is very similar to the way you would define it for, e.g., the C# language. The only major difference is that instead of supporting a general IContextAction interface, you need to inherit from a special class. (And even that is optional - `XmlContextAction` is simply an intermediate class that ensures you pick the right data provider.)

Now that you have a context action, it becomes a matter of defining the familiar Text, `IsAvailable()` and `ExecutePsiTransaction()` members. The `IsAvailable()` method is the place where you would check that the tag you're on is empty. For example:

```csharp
var tag = DataProvider.FindNodeAtCaret<IXmlTag>();
if (tag == null || tag.IsEmptyTag || !XmlTagUtil.CanBeEmptyTag(tag))
  return false;
```

Manipulation of XML tokens is very important if you want to do fine-tuning of your XML-related actions. To get at the token at the caret location, for example, you would write:

```csharp
var token = DataProvider.FindNodeAtCaret<IXmlToken>();
```

Now, you can, for example, call `GetTokenType()` to determine the type of the token you're on. Keep in mind that `IXmlToken` is inherited by many other interfaces such as e.g., `IXmlIdentifier`. After determining the type of token, you can cast to the right type to get at the token-specific properties and methods.

Here's a brief example: suppose you want to remove all inner nodes of a particular tag. To do this, you would take the tag's header node and then take its next sibling. Then you would also take its last child. Finally, you would use the `ModificationUtil` class in order to delete all children within that range:

```csharp
var header = tag.HeaderNode;
var first = header.NextSibling;
if (first != null)
{
  var last = tag.LastChild;
  ModificationUtil.DeleteChildRange(first, last);
}
```

The `ModificationUtil` class contains several methods for adding and removing elements from the XML token tree.

### Example: working with an XML attribute

Here's a simple example: let's suppose that you have a tag, and you want to add an attribute to it - for example, an attribute for an XML namespace. The first thing you do is acquire an `XMLElementFactory`.

```csharp
IXmlElementFactory factory = XmlElementFactory.GetElementFactory(tag);
```

Then, you can synthetically prepare the attribute declaration:

```csharp
string text = String.Format("{0}=\"{1}\"", namespacePrefix, namespaceUrl);
```

With the declaration, you can use the factory to create a whole XML file. This may sound 'heavyweight', but is in fact a convenient way to get the structures parsed and ready for subsequent insertion.

```csharp
IXmlFile file = factory.CreateFile(provider.PsiServices, provider.PsiModule,
                                   "<tag " + text + "/>", true);
```

Finally, you can get at the attribute in the created file and insert it into the tag you're trying to change:

```csharp
IXmlAttribute att = file.InnerTags.First().GetAttributes().First();
tag.AddAttributeBefore(att, null);
```

## HTML

Work with HTML files is a bit different than XML. The first thing to note is that, unlike with XML, an HTML context action, for instance, would have no special base class of its own. Rather, making an HTML-related context action is simply a matter of implementing the `IContextAction` interface. The only difference is that this context action would take an `IWebContextActionDataProvider<T>` as a parameter.

The type parameter `T` has to be either of type `IHtmlFile` or one of its inheritors - `IAspFile` and `IRazorFile`. Here's an outline of a context action for HTML files:

```csharp
[ContextAction(Group = "HTML",
  Name = "Specify Id",
  Description = "Creates an 'id' attribute for the selected tag of an XML document")]
public class SpecifyIdHtmlContextAction : IContextAction, IBulbItem
{
  private readonly IWebContextActionDataProvider<IHtmlFile> provider;

  public SpecifyIdHtmlContextAction(IWebContextActionDataProvider<IHtmlFile> provider)
  {
    this.provider = provider;
  }
}
```

Now, you'll notice that the code for creating an id-inserting context action is very similar to what we did for XML. The only differences are in the types of tags and factories that we are using:

```csharp
public bool IsAvailable(IUserDataHolder cache)
{
  var tag = provider.FindNodeAtCaret<IHtmlTag>();
  if (tag != null)
  {
    var idAtt = tag.Attributes.FirstOrDefault(a => a.AttributeName.Equals("id"));
    if (idAtt == null)
    {
      Tag = tag;
      return true;
    }
  }
  return false;
}
```

Now, when it comes to modifying the document tree, with Web files there is one difference: you need to explicitly create the transaction before modifying the HTML file. Here's a piece of code that, just like in the XML example, inserts the `id` attribute into a tag:

```csharp
public void Execute(ISolution solution, ITextControl textControl)
{
  var psiServices = solution.GetComponent<IPsiServices>();
  using (new PsiTransactionCookie(psiServices, DefaultAction.Commit, Text))
  {
    var factory = HtmlElementFactory.GetInstance(Tag.Language);
    var dummy = factory.CreateHtmlTag("<tag id=\"\"/>", Tag);
    Tag.AddAttributeBefore(dummy.Attributes.First(), null);
  }
}
```

As you can see, the principle for creating the attribute is the similar to what we had before, with the difference that instead of creating a whole file, we can simply create a tag. From then on, getting the first attribute and inserting it into our element is easy.
