[//]: # (title: Get a Tree Node by Reference)

**What you should know beforehand:**
* [PSI](NavigateCode.md#psi-basics)
* ShellLocks

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [PsiExtensionMethods.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/PsiExtensionMethods.cs)
* [NodeUnderCaretDetector.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/NodeUnderCaretDetector.cs)

This is useful in case you need to navigate to an element that is referenced by a particular tree node (e.g., from a usage to a declaration).

So, the task is to obtain the [reference](NavigateCode.md#psi-basics) you need. For example, we want to create a navigation from a particular expression to a declaration (the example is quite useless as it duplicates ReSharper's functionality, nevertheless, it perfectly illustrates the usage of references).

Let's make this as an extension method for `ITreeNode`:

```csharp
[CanBeNull]
public static IEnumerable<IDeclaredElement> GetReferencedElements(this ITreeNode node)
{
    var result = new List<IDeclaredElement>();
    var parentExpression = node.GetParentOfType<IReferenceExpression>();
    if (parentExpression == null) return null;            
                          
    var references = parentExpression.GetReferences();            
 
    foreach (var reference in references)
    {
        var declaredElement = reference.Resolve().DeclaredElement;
        if (declaredElement != null)
            result.Add(declaredElement);
    }
 
    return result.Count == 0 ? null : result;
}
```

Now, it can be used to navigate to the first referenced element of the tree node under the caret (it would be always a declaration):

```csharp
private IShellLocks _shellLocks;
private NodeUnderCaretNavigator _navigator;
 
// ...
 
public void NavigateToFirstReferencedElement()
{
    _shellLocks.ExecuteOrQueueReadLock(_lifetime, "NavigateByReference", () =>
    {
        var node = _navigator.GetTreeNodeUnderCaret();
        var referencedElements = node.GetReferencedElements();
        var element = referencedElements?.FirstNotNull();
        element?.Navigate(true);
    });
}
```

## Notes
* We use the `GetTreeNodeUnderCaret()` method of the `NodeUnderCaretNavigator` class shown in [Get a Tree Node Under Caret](GetTreeNodeUnderCaret.md).
* We use the `GetParentOfType` method from [Use Manual Navigation](UseManualNavigation.md) to obtain an [IReferenceExpression](NavigateCode.md#psi-basics) instance.
* The `GetReferences` method returns the list of references.
* The `IReference` interface provides the `Resolve` method that allows getting a declared element from a reference.
* Note that to prevent conflicts with user input during the navigation, we must obtain the read lock before calculating the node under the caret. This is done by means of the special method of the `IShellLocks` interface: `_shellLocks.QueueReadLock(...)`.