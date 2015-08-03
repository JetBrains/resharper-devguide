---
---

# Manually Navigating a Syntax Tree

Manually walking a syntax tree is the lowest level means of navigation, and involves walking the nodes using the properties defined on `ITreeNode`.

> **NOTE** Generally speaking, you should prefer to use [strongly typed navigation](StronglyTypedNavigation.md) or the visitor pattern to walk and process the tree. Failing that, you should use the `TreeNodeExtensions` helper methods that have implemented the boilerplate tree walking code for you.

* Table of Contents
{:toc}

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

> **NOTE** While you do need to recursively walk the structure (walk a child node's children, then it's children, etc.), beware of an implementation that uses recursion to accomplish this. Recursion can run into problems with a very deeply nested syntax tree. It is better to iteratively walk the tree than to use recursion.
>
> Even better, you should avoid writing your own methods for walking the tree at all, and make use of the `TreeNodeVisitor` or `IRecursiveElementProcessor` patterns that ReSharper implements. Failing that, use the `TreeNodeExtensions` helper methods.

