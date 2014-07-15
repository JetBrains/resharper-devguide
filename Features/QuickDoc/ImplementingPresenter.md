# Implementing IQuickDocPresenter

Once a provider has identified an element that it can generate documentation for, it creates an instance of `IQuickDocPresenter`, which is responsible for generating the documentation. It has the following methods:

```cs
public interface IQuickDocPresenter
{
  string GetHtml(PsiLanguageType presentationLanguage);
  [CanBeNull] string GetId();
  [CanBeNull] IQuickDocPresenter Resolve(string id);
  void OpenInEditor();
  void ReadMore();
}
```

The **`GetId`** method returns the ID of the current code element being displayed. This may be an XML Doc Comment ID, a simple ID, or even `null`. It is intended to be used when displaying the name of the code element, or as diagnostics in the case of failure. For example, if `GetHtml` returns null, the ID is used in a message of "no documentation available for {ID}".

**`Resolve`** will create a new instance of `IQuickDocPresenter` to allow navigation from one code element to another. For example, a C# QuickDoc window for a method can show parameter and return types as hyperlinks. Clicking these will change the content of the QuickDoc view to that of the new type. Typically, this method is very similar to `IQuickDocProvider.Resolve` and some of the default providers actually call a `Resolve` overload on their creating provider. Alternatively, if it's a CLR based language, it can be deferred to an instance of `QuickDocTypeMemberProvider`, which will treat the passed in code element ID as a CLR XML Doc Comment ID. If the presenter doesn't support navigation, or cannot navigate to the item, it can return `null`, in which case, it should not add hyperlinks to the HTML.

The **`OpenInEditor`** and **`ReadMore`** methods are called in response to the "go to" or "read more" hyperlinks added to the default HTML template. When the user clicks the hyperlink, the current presenter's `Resolve` method is called to find the presenter for the target, and the new presenter's `OpenInEditor` or `ReadMore` method is called. `OpenInEditor` is also called if the user holds down `Ctrl` when clicking a type definition hyperlink.

`ReadMore` is expected to open further documentation. It can do this with a simple call to `Process.Start`, passing in a URL to spawn the system browser, or using the `HelpSystem` component to display product or MSDN documentation (again extensible by implementing `IShowHelp`).

The most important method is **`GetHtml`**. The string returned from this method is displayed verbatim in a web browser control. If the current code element doesn't have any documentation, or the documentation cannot be created, this method should return `null`, in which case a fallback message, such as "no documentation available for {ID}" is displayed.

>**Note** It is the responsibility of the `IQuickDocPresenter` to ensure the HTML is properly sanitised before display (see [RSRP-186295](http://youtrack.jetbrains.com/issue/RSRP-186295) for a great reason why!). Using the helper classes mentioned below can help with this, particularly `XmlDocPresenterUtil.EscapeHtmlString`

ReSharper provides a couple of classes to help build the HTML - **`XmlDocHtmlUtil`** and **`XmlDocHtmlPresenter`**. Typically, the presenter will generate the HTML by calling `XmlDocHtmlPresenter.Run` which in turn calls methods on `XmlDocHtmlUtil`, such as `XmlDocHtmlUtil.BuildHtml`, although the presenter is free to call `XmlDocHtmlUtil` directly, or create the HTML itself. It is, however, recommended to use the helper classes).
