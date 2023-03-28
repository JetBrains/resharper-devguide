[//]: # (title: Use Manual Navigation)

**What you should know beforehand:**
* [PSI](NavigateCode.md#psi-basics)
* ShellLocks

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [PsiExtensionMethods.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/PsiExtensionMethods.cs)
* [NodeUnderCaretDetector.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/NodeUnderCaretDetector.cs)

In some cases it might be useful to manually navigate the syntax tree (relatively to a particular tree node). For this purpose, any tree node has a number of properties and methods, like:
* `Parent`: returns a parent tree node;
* `Children`: returns `IEnumerable` with children nodes;
* `NextSibling`, `PrevSibling`: next and previous sibling tree nodes;
* lots of them.

For example, let's create an `ITreeNode` extension method that walks up the syntax tree and finds (if there are any) a parent tree node of a particular type:

```csharp
[CanBeNull]
public static T GetParentOfType<T>(this ITreeNode node) where T : class, ITreeNode
{
    while (node != null)
    {
        var typedNode = node as T;
        if (typedNode != null)
            return typedNode;

        node = node.Parent;                                                                
    }
    return null;
}
```

For example, we can use it to navigate to the method declaration while standing anywhere inside a method:

```csharp
private IShellLocks _shellLocks;
private NodeUnderCaretNavigator _navigator;
  
...
  
public void NavigateToParentMethod()
{
    _shellLocks.ExecuteOrQueueReadLock(_lifetime, "NavigateToParent", () =>
    {
        var node = _navigator.GetTreeNodeUnderCaret;
        var parentMethod = node.GetParentOfType<IMethodDeclaration>();
        parentMethod?.NavigateToTreeNode(true);
    });
}
```

## Notes
* We use the `GetTreeNodeUnderCaret()` method of the `NodeUnderCaretNavigator` class shown in [Get a Tree Node Under Caret](GetTreeNodeUnderCaret.md).
* Note that to prevent conflicts with user input during the navigation, we must obtain the read lock before calculating the node under the caret. This is done by means of the special method of the `IShellLocks` interface: `_shellLocks.QueueReadLock(...)`.
