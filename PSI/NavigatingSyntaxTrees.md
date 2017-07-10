---
title: Navigating syntax trees
---

Every node in the syntax tree implements `ITreeNode`, as well as interfaces that better reflect the node's type and purpose, such as `IComment` or `IMethodDeclaration`. The `ITreeNode` interface includes members pertinent to navigating and processing the tree. These members are shown below (other members removed for brevity):

```csharp
public interface ITreeNode
{
  ITreeNode Parent { get; }
  ITreeNode FirstChild { get; }
  ITreeNode LastChild  { get; }
  ITreeNode NextSibling { get; }
  ITreeNode PrevSibling { get; }

  // [...snip...]
}
```

There are several ways of navigating the tree.

* [Manually walking the tree](SyntaxTrees/ManualNavigation.md), using the interface members shown above. This is the least recommended, requiring boilerplate tree walking code that is time consuming and easy to get wrong.
* Use the `TreeNodeExtensions` helper methods to manually walk the tree.
* Use [strongly typed navigation](SyntaxTrees/StronglyTypedNavigation.md), using accessors on derived `ITreeNode` implementations, and the strongly typed "navigator" classes.
* Process the tree using the visitor pattern or the `IRecursiveElementProcessor` set of classes.
* Finding child or parent nodes of a specific type.

