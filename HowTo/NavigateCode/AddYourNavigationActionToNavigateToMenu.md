---
title: Add a Navigation Action to the 'Navigate to' Menu
---

**What you should know beforehand:**
* [PSI](/HowTo/NavigateCode/NavigateCode.md#psi-basics)

**Examples ([?](HowTo.md#sample-solution)):**
* [NavigateToCtorProvider.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/NavigateToCtorProvider.cs)

If you want your navigation to appear in ReSharper's **Navigate To** menu ([context navigation](NavigateCode.md)), you should create an action that implements the `INavigateFromeHereProvider` interface.

![add-navigate-to-action](add-navigate-to-action.png)

For example, let's create a context-dependent navigation that navigates to a class constructor. The initial caret position can be anywhere inside that class.

```csharp
[ContextNavigationProvider]
public class NavigateToCtorProvider : INavigateFromHereProvider
{
    public IEnumerable<ContextNavigation> CreateWorkflow(IDataContext dataContext)
    {
        var node = dataContext.GetSelectedTreeNode<ITreeNode>();
        var typeDeclaration = node?.GetParentOfType<IClassDeclaration>();            
        var constructor = typeDeclaration?.ConstructorDeclarations.FirstNotNull();
 
        if (constructor != null)
        {
            yield return new ContextNavigation("Constructor", null, NavigationActionGroup.Other, () =>
            {
                constructor.NavigateToTreeNode(true);
            });
        }                       
    }
}
```

## Notes

* The class must implement the `INavigateFromHereProvider` interface and be marked with the `ContextNavigationProvider` attribute.
* `GetSelectedTreeNode<ITreeNode>()` method of `IDataContext` return the tree node under the current caret position.
* We use the `GetParentOfType` method from [Use Manual Navigation](UseManualNavigation.md) to obtain the `IClassDeclaration` node.
* The `IClassDeclaration` class provides the `ConstructorDeclarations` property that returns an enumerable with class constructors.
* A created instance of the `ContextNavigation` class describes the navigation feature:
    * `"Constructor"`: the name of the item that appears in the **Navigate to** context menu.
    * `null`: action ID.
    * `NavigationGroup.Other`: defines the location in the **Navigate to** menu where the action should be placed.
    * The last one argument is the action that is run once the **Constructor** item is selected in the **Navigate to** menu.
* Note that ReSharper automatically decides whether to display the item in the menu or not.
