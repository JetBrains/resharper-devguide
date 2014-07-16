# Implementing IQuickDocProvider

The `IQuickDocProvider` interface provides two methods:

```cs
public interface IQuickDocProvider
{
  bool CanNavigate(IDataContext context);
  void Resolve(IDataContext context, Action<IQuickDocPresenter, PsiLanguageType> resolved);
}
```

**`CanNavigate`** is called to see if the provider can generate documentation based on the given `IDataContext`. If it returns true, no other providers are checked. If it returns false, the `QuickDocManager` continues to other providers. 
    
The provider queries the passed in `IDataContext` to find data that it needs to provide documentation. This might be using `DataConstants.DOCUMENT` and `DataConstants.DOCUMENT_OFFSET` to get the current document and offset, in order to get the PSI tree node at the text caret location, or it might be using `DataConstants.DECLARED_ELEMENTS` to get the `IDeclaredElement` at the text caret. The requirements here depend on what the provider wants to document. (See `IDataContext` for more details.)

**`Resolve`** is called by the `QuickDocManager` when initially displaying the documentation (the name makes more sense when dealing with navigation from one presenter to another). The `IDataContext` is passed in, from which the provider should extract whatever data is required in order to be able to display documentation, and this data is used to create an instance of `IQuickDocPresenter`. The provider then retrieves the `PsiLanguageType` of the item being documented, and calls the passed in `resolved` action. (This action, provided by `ShowQuickDocAction`, is responsible for creating and managing the popup window and the web browser used to host the HTML).

>**Note** Retrieving the language can be done using the document from the `IDataContext`, and passing it, together with instances of `DocumentManager` and `ISolution` (injected into the constructor of your component) to the `PresentationUtil.GetPresentationLanguageByContainer` method:
>
> ```cs  
> var document = context.GetData(DataConstants.DOCUMENT);
> if (document != null)
>   projectFile = myDocumentManager.TryGetProjectFile(document);
>
> var defaultLanguage = PresentationUtil.GetPresentationLanguageByContainer(projectFile, mySolution);
> resolved(presenter, defaultLanguage);
> ```
  
You need to decorate your `IQuickDocProvider` class with `QuickDocProviderAttribute`, and pass a priority into the constructor. The list of providers maintained by `QuickDocManager` is an ordered list, with the lowest priority being at the start of the list (any new items being added to the `QuickDocManager`'s `IViewable<IQuickDocProvider>` are inserted in the correct position in the list). The majority of the default providers have a priority of `0`. You only need to provide a higher or lower priority if the items you're intending to document are also handled by the default providers, and you wish to override or provide a fallback.

```cs
public class QuickDocProviderAttribute : SolutionComponentAttribute
{
	public QuickDocProviderAttribute(int priority);
	public int Priority { get; }
}
```

>**Note** If an `IQuickDocProvider` or `IQuickDocPresenter` is displaying the QuickDoc information for an instance of `IDeclaredElement`, they should not store the declared element as a field, as the presenter can last longer than the lifetime of the declared element. E.g. if the QuickDoc window is pinned, and the text of the file is edited, the declared element may become invalid.
>
> Instead, the presenter should create a `DeclaredElementEnvoy`, which can both cache the presentation of the element, and recreate a valid element, with a call to `GetValidDeclaredElement`.
