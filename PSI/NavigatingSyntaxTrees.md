---
---

# Navigating syntax trees

Every node in the syntax tree implements `ITreeNode`. The interface includes members pertinent to navigating and processing the tree. These members are shown below (other members removed for brevity):

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

* Manually walking the tree, using the interface members shown above
* Use the helper methods to manually walk the tree
* Use [strongly typed navigation](SyntaxTrees/StronglyTypedNavigation.md), using accessors on derived `ITreeNode` implementations, and the strongly typed "navigator" classes
* Process the tree using the [visitor pattern or the `IRecursiveElementProcessor` set of classes](SyntaxTrees/RecursiveNavigation.md)
* Finding child or parent nodes of a specific type

## Manually navigating the tree

Manually walking the tree is the lowest level means of navigation, and involves walking the nodes using the properties shown above.

> **NOTE** Generally speaking, you'll want to use one of the other, less low level, navigation techniques in this section to navigate around the tree. However, it is useful to know how to navigate the tree in order to understand how the tree is structured.

### Walking up the tree

You can walk up the tree using the `Parent` property. This is very useful for finding the context that the current node is in - for example, you can walk the `Parent` property of a method invocation to see if the invocation is happening inside a method declaration that is async or not.

To walk the `Parent` property, you will want to write code like this:

```csharp
while(currentNode != null)
{
  // Process node
  currentNode = currentNode.Parent;
}
```

### Walking sibling nodes

Since a file is parsed into a tree structure, nodes have siblings as well as children. For example, a C# file can have a class declaration that contains multiple methods. Each method is represented by an `IMethodDeclaration` node. Intuitively, each method is a child of the class, and therefore are siblings, and no method is higher or lower in the tree than any other.

> **NOTE** Even though the method declarations will be siblings, they are not going to be direct siblings. The syntax tree is a full-fidelity representation of the C# file, which means the whitespace and comments between the method declarations are also added as nodes in the tree, and these will be siblings to the method declarations.

You can walk the siblings with code like this:

```csharp
while(currentNode != null)
{
  // Process node
  currentNode = currentNode.NextSibling;
}
```

Obviously the code to walk the sibling chain in reverse is just the same, but using the `PrevSibling` property.

### Walking down the tree

Walking down the tree is very similar to walking the sibling chain. Again, since this is a tree structure a single node can have multiple children. For example, a C# file's list of `using` statements is represented by an `IUsingList` node, and this will have multiple children - multiple instances of `IUsingNamespaceDirective` (with each child again separated by whitespace nodes).

To walk the immediate children of a node, you'll want to write code like this:

```csharp
public void WalkChildren(ITreeNode root)
{
  var child = root.FirstChild;
  while(child != null)
  {
    // Process child node
    child = child.NextSibling;
  }
}
```

Note that we start with `FirstChild` but then walk that child's siblings. To walk the children in reverse order, start with `LastChild`, and walk `PrevSibling`.

If you want to walk all descendants of a node, simply recursively walk a child node's children.

> **NOTE** Beware of recursively walking the descendants of a node. Recursion can run into problems with a very deeply nested syntax tree. It is better to iteratively walk the tree than to use recursion.
>
> You should avoid writing your own methods for walking the tree, and make use of the [`TreeNodeVisitor` or `IRecursiveElementProcessor`](SyntaxTrees/RecursiveNavigation.md) patterns that ReSharper implements.

