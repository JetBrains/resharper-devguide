---
---

# Composite Node Types

The interior nodes of the tree have a node type that typically derives from `CompositeNodeType`. ReSharper doesn't have any strict requirements for the node type of an interior tree node, unless the custom language implements tree node caching, in which case, the node type must implement the `ICompositeNodeType` interface:

```csharp
public interface ICompositeNodeType
{
  CompositeElement Create();
}
```

As with `TokenNodeType.Create`, `ICompositeNodeType.Create` returns an implementation of `ITreeNode` that uses ReSharper's own base classes. The `Create` method is only used by the parser (via the `TreeElementFactory.CreateCompositeElement` static method) to create an `ITreeNode` from a known node type.

> **NOTE** ReSharper has a couple of implementation details that require a custom language uses ReSharper's base classes to implement `ITreeNode`, specifically downcasting the root `IFile` node to `FileElementBase` after it's been created. Simiarly, `ICompositeNodeType.Create` returns a `CompositeElement` derived class.

If a custom language is using ReSharper's base `ITreeNode` classes, it should also use `CompositeNodeType` as the base class for all interior tree node types. This base class has no additional functionality over `NodeType`, and simply provides an abstract method to create the `CompositeElement` base tree node instance.

## Composite node type hierarchies

The class hierarchy for composite node types is very flat. A custom language should create its own composite node type base class. It adds no functionality, but acts as a common base class for all derived composite node types for the language. For example, CSS creates `CssCompositeNodeType`:

```csharp
public abstract class CssCompositeNodeType : CompositeNodeType
{
  protected CssCompositeNodeType(string s, int index)
    : base(s, index)
  {
  }
}
```

The base class simply takes in a string identifier, such as `"ID_SELECTOR"`, used only for diagnostics and testing, and an index to uniquely identify the node type within the language. This index must not clash with any existing indexes used for that language - token or composite node types.

All composite node types of the custom language should be created as derived classes of this language-specific composite node type. The only extra functionality required in these derived classes is to provide an implementation for the `Create` method, by creating an `ITreeNode` that represents that tree node. For example:

```csharp
public class ElementType
{
  // ...

  private class RULESET_INTERNAL : CssCompositeNodeType
  {
    RULESET_INTERNAL(int index)
      : base("RULESET", index)
    {
    }

    public override JetBrains.ReSharper.Psi.ExtensionsAPI.Tree.CompositeElement Create()
    {
     return new Ruleset();
    }
  }

  // ...
}
```

See the section on [Creating Node Types](CreatingNodeTypes.md) for an explanation of the private nested class structure.

The `Create` method will create an `ITreeNode` instance. There is no need to pass any parameters - the position of the element is implied by its predecessor nodes (and cached and invalidated if the tree is altered). Given the start and end offset, it's possible to get the total text for the node and all associated child nodes.

There is also no need to pass details about the content of the node, as this information is made available via child nodes. For example, a node that represents a C# assignment operation will have children that represents the left hand side identifier, the equals sign and the right hand side expression. The left hand side node can have further child nodes to expose qualifications (`this.value = …`) or can be a leaf node that exposes the identifier directly.
