---
---

# Node Types

Each node in a given PSI tree implements `ITreeNode`, which has a property called `NodeType` that identifies the type of the node. The value is a singleton instance that derives from the abstract base class `NodeType`. Leaf elements in the tree have a node type that further derives from the `TokenNodeType` abstract base class, while interior tree elements derive from `CompositeNodeType`.

The `NodeType` class is the abstract base class for all node types:

```csharp
public abstract class NodeType
{
  private readonly string myString;
  private readonly int myIndex;
  protected NodeType myBaseNodeType;

  // "index" must be the unique identifier of the NodeType inside language
  protected NodeType(string s, int index)
  {
    myString = s;
    myIndex = index;
  }

  public NodeType BaseNodeType
  {
    get { return myBaseNodeType; }
  }

  internal int Index
  {
    get { return myIndex; }
  }

  public override string ToString() { return myString; }
}
```

The `Index` property is an integer that uniquely identifies the node type within the language. It is used by the language's implementation of `ILanguageCacheProvider`, to allow caching and retrieving a node type, and identifying it by the index.

The string value passed to the constructor is only used for diagnostic purposes and testing. The value can be anything, but should be unique enough to identify the node type. For example, given a node type that has a static representation, such as the `&&` operator or the `return` keyword, C# uses `ANDAND` and `RETURN_KEYWORD`. For nodes types that don't have a static representation, such as identifiers, C# doesn't use the changeable representation, but will use a static value such as `IDENTIFIER` or `NEW_LINE`, etc.

The `BaseNodeType` property is rarely used, but allows one node type to be treated like another node type. It's only used in JavaScript's WinRT support, where a WinRT expression can be treated either as a WinRT expression, or a plain JavaScript expression. For the majority of uses, it can be ignored.

See the sections on [Token Node Types](NodeTypes/TokenNodeTypes.md), [Composite Node Types](NodeTypes/CompositeNodeTypes.md) and [Creating Node Types](NodeTypes/CreatingNodeTypes.md).
