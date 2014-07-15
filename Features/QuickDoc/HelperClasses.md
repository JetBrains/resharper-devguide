# Presenter helper classes

ReSharper provides a couple of classes to help build the HTML - **`XmlDocHtmlUtil`** and **`XmlDocHtmlPresenter`**. Typically, the presenter will generate the HTML by calling `XmlDocHtmlPresenter.Run` which in turn calls methods on `XmlDocHtmlUtil`, such as `XmlDocHtmlUtil.BuildHtml`, although the presenter is free to call `XmlDocHtmlUtil` directly, or create the HTML itself. It is, however, recommended to use the helper classes).

### XmlDocHtmlUtil

Since the presenter has to create the entire HTML that is displayed, it should at the very least use the standard HTML style template that is defined as a public constant in `XmlDocHtmlUtil.QUICK_DOC_HTML_STYLE`:

```cs
return "<html><head>\n" +
  "<META http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\">\n" +
  XmlDocHtmlUtil.QUICK_DOC_HTML_STYLE +
  "  </head><body>\n" +
  body +
  "</body></html>";
```

However, doing this doesn't include the standard scripts for showing and hiding sections, or the content for the "go to" and "read more" links. To get this, call `XmlDocHtmlUtil.BuildHtml`, passing in a function that will insert the HTML body into a given `StringBuilder`, and also a `NavigationStyle` enum to state if you want "go to", "read more", both or none:

```cs
BuildHtml(sb => sb.Append(body), NavigationStyle.All);
```

There are a number of other helper methods:

* `Hyperlink` adds an `<a>` tag to link to the passed in code element ID (not just XML Doc Comment ID - this ID is passed to `IQuickDocPresenter.Resolve` when the link is clicked). The hyperlink is created in the form `cref:ID`, which explains the mention of `cref` elsewhere in the API.

    There are two overloads, one take a simple tooltip parameter, which is frequently set to the fully qualified typename of the code element, and the other takes a class whose properties are used to add attributes to the `<a>` tag.

    This can be used with anonymous types. For example, CLR attributes get the "ext" class added to indicate external annotation attributes:
    
    ```cs
    var htmlFragment = Hyperlink(name, id, new {
      @class = "ext",
      title = fullyQualifiedName    })
    ```
        
* `ProcessCRef` will also create a hyperlink, by calling `Hyperlink`. It will use the `linktext` parameter passed in as the text, if available. If not, it will try to use the `cref` parameter as an XML Doc Comment ID to resolve a CLR `IDeclaredElement`, and use the name as the link text and fully qualified name as a tooltip. Failing that, it strips any colon-delimited prefix (e.g. "M:") from the `cref` parameter to use as the link text.
* `OpenCollapsedRegion` creates a new region (by inserting a `<div>`) that is collapsed by default. More content can be added to the `StringBuilder` to populate the region, and `CloseCollapsedRegion` will close the `div`. It is used to include collapsed region in the documentation summary.
* `OpenAttributesRegion`/`CloseAttributesRegion` is very similar to `OpenCollapsedRegion`/`CloseCollapsedRegion` except it has different styling, and shows up in the top left of the QuickDoc window. It is used to show the attributes for CLR types and type members.

### XmlDocHtmlPresenter

`XmlDocHtmlPresenter` is a higher-level API for generating HTML QuickDoc content. It expects an XML node based on the CLR XML Doc Comment format. However, the API is not restricted to CLR XML Doc Comments. The XML is used to create structured HTML, and can be used to generate JavaScript function or CSS type property documentation just as easily as C# type member documentation. Several of the default providers create a new xml node in order to generate documentation.

The entry point to the API is the `Run` method:

```cs
var html = XmlDocHtmlPresenter.Run(node, module, element,
  language, navigationStyle, processCRef)
```

The parameters are:

* `node` - the XML node representing the documentation.
* `module` - an instance of `IPsiModule` that is passed through untouched to the `processCRef` delegate.
* `element` - the `IDeclaredElement` instance that the documentation is for. If specified, it's used to create the code element name header information. If null or empty, the name of the code element comes from the XML node.
* `language` - the `PsiLanguageType` used to format the `IDeclaredElement` name.
* `navigationStyle` - "go to", "read more", both or none
* `processCRef` - a delegate to handle what happens when references to other code elements should be rendered. Usually defers to `XmlDocHtmlUtil.ProcessCRef` to create a hyperlink.

`Run` calls `XmlDocHtmlUtil.BuildHtml` to create the skeleton HTML, and provides its own implementation of the `appendTextBody` delegate to populate the HTML body content. This implementation processes the XML node, and formats the HTML appropriately.

#### XML node format

The `Run` methods understands the standard CLR XML Doc Comment format, and can also handle unknown elements. 

Generally, the root node is the `member` node, and is used to generate a name heading for the documentation.

All child nodes of the `member` node are converted into sections in the generated HTML. Each section has a title, such as "Remarks" or "See Also", and the content of each XML node is converted and displayed in the section.

>**Note** the root node of the XML will only be processed if an `IDeclaredElement` is passed in, otherwise the child nodes are immediately processed instead. While it is assumed that the root node is the `member` node, this is not a requirement.

The following elements are recognised:

* `member` - if an `IDeclaredElement` is specified, the per-language `IXmlDocHeaderPresenter` interface is used to format the name of the `IDeclaredElement`. If no element is passed in, the `name` attribute of the `member` tag is passed to the `processCRef` delegate (which typically defers to `XmlDocHtmlUtil.ProcessCRef`, which creates a hyperlink in the form `cref:id`). As such, the `name` attribute should be the CLR XML Doc Comment ID of the element, or a similar ID understood by the QuickDoc presenter.

    After processing the `member` name, the node's children are processed, and converted into top level "sections" in the HTML document.
* `include` will include the content of another file.
	* The `file` attribute provides either the fully qualified path name of the file, or a filename relative to the source file containing the declaration of the `IDeclaredElement`. 
	* Once the file has been read, the `path` attribute is used as an XPath statement to retrieve XML nodes which are in turn processed into HTML
* `example`, `summary`, `value`, `remarks`, and `return` are simply converted into top level HTML sections, with their child nodes processed as content.
* `exception` and `permission` are formatted as a table in the HTML, with the child nodes providing the content, and the `cref` attribute formatted into a hyperlink with the `processCRef` delegate. The `cref` attribute should be an ID understood by the presenter.
* `param` and `typeparam` are also formatted into a HTML table, but the `name` attribute is passed to `processCRef` for the name.
* `br` inserts a `<br>` HTML element.
* `c` inserts the content of the XML node verbatim into a `<code>` HTML block. The inner text of the XML node is escaped with `XmlDocPresenterUtil.EscapeHtmlString` first
* `code` also inserts the content of the XML node as a pre-formatted `<pre>` element. Some cleansing is applied to the pre-formatted text, to normalise indenting, newlines and escape HTML strings.
* `para` processes its child nodes, and wraps the resulting content in a `<p>` element.
* `paramref` and `typeparamref` format the `name` attribute in a HTML `<var>` element.
* `seealso` passes the `cref` attribute to `processCRef`, and the resulting hyperlink is added to the HTML. `cref` should be an ID understood by the presenter.
* `see` also passes the `cref` attribute to `processCRef` and appends the output hyperlink. It also supports the `langword` and `title` attributes.
* `list` looks for the `type` attribute to see what kind of list is to be created. Accepted values are `bullet` and `number`. If no value is specified, the list is formatted as a table.
	* if the `listheader` child element is specified, its `term` and `description` elements are processed (to handle links, code or `seealso`, etc.) and added to a `<tr>`, `<ul>` or `<ol>` element, depending on the type of the list.
	* Each `item` node's `term` and `description` elements are added to the list, as either `<tr>` or `<li>` elements.

If a top-level element (i.e. a direct child of the root node, or the `member` node) is not one of the elements in the above list, a new section is created for it. The element name is capitalised to make a section name, and the child nodes of the element are processed as above. This means an unknown element can contain known elements such as `code` and `seealso`.

If the unknown element is not top-level, then the XML is added to the text of the description. It is not rendered as HTML, but as plain text, with all tags and attributes being encoded to display as-is.

#### `IXmlDocHeaderPresenter`

If an `IDeclaredElement` is passed in, and the member header is to be rendered, the rendering is deferred to a per-language implementation of `IXmlDocHeaderPresenter`. ReSharper provides per-language implementations for C#, VB and CSS. All other languages share the `CommonXmlDocHeaderPresenter`.

If you are implementing your own language, and you do not wish to use the `CommonXmlDocHeaderPresenter`, you should also implement `IXmlDocHeaderPresenter`. The interface is simple:

```cs
public interface IXmlDocHeaderPresenter{  void Present(StringBuilder header, IDeclaredElement declaredElement, IPsiModule module);}
```

Calling `Present` will format the `declaredElement` into the given `StringBuilder`.

The default implementation in `CommonXmlDocHeaderPresenter` will use `DeclaredElementPresenter.Format` to format the name (this defers to the language's `IDeclaredElementPresnter`, another extension point for implementing your own language). It then wraps the name in `<dl>` and `<dt>` HTML tags.

The C# implementation overrides this default, and while it too calls `DeclaredElementPresenter.Format`, it also adds formatting for type parameters, method parameters and attributes on the type, type member or parameters, as well as annotations for where the element is declared ("in class", "in assembly", etc.)

If you wish to implement support for your own language that uses CLR attributes, you can provide a per-language implementation of `IHtmlAttribtuePresenter`, and use the `HtmlAttributesPresenterBase` class to provide a standard formatting of CLR attributes that are initially hidden and expand only when the user clicks on them.