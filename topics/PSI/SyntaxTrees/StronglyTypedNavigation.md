[//]: # (title: Strongly Typed Navigation)

While it's possible to navigate a syntax tree using just the parent, sibling and child nodes, it's a rather low level approach to the problem. If your current node is a class declaration, it's a lot of leg work to get at all of the method declarations. Similarly, from a method declaration, it's not convenient to walk the `Parent` nodes looking for the class declaration.

Fortunately, ReSharper provides strongly typed navigation to make this a lot more convenient and intuitive.

## Navigating down the tree

Navigating down to child nodes is made very easy by derived interfaces of `ITreeNode`. All nodes in the tree implement `ITreeNode`, and also derive from it to provide a strongly typed API. For example, C# method declarations implement `IMethodDeclaration`, which itself derives from `ITreeNode`. If we take a look at some of the members:

```csharp
public interface IMethodDeclaration : ITreeNode
{
  IAttributeSectionList AttributeSectionList { get; }
  ICSharpIdentifier NameIdentifier { get; }
  ITypeParameterOfMethodList TypeParameterList { get; }
  // [...snip...]
}
```

 >  This example over-simplifies the real implementation of `IMethodDeclaration`. The real definition applies the Interface Segregation Principle and splits out more interfaces that can be reused, such as `ICSharpParametersOwnerDeclaration` and `ICSharpModifiersOwnerDeclaration` to allow other nodes that can also have parameters, or modifiers, or attributes, etc., to share interfaces. The intent here is to discuss the strongly typed accessors, rather than document `IMethodDeclaration`.
 >
 {type="note"}

Each of these properties is a strongly typed accessor to get to the `ITreeNode` that represents the attributes on the method, or the node that represents the method's name. Internally, the nodes are found by walking down the tree, looking for nodes of a particular type. They are then downcast to the strongly typed, `ITreeNode` derived interfaces.

### Collections

Similarly, collections can be represented. Let's take a look at a simplified class declaration:

```csharp
public interface IClassDeclaration : ITreeNode
{
  TreeNodeCollection<IMethodDeclaration> MethodDeclarations {get;}
  TreeNodeEnumerable<IMethodDeclaration> MethodDeclarationsEnumerable {get;}
  // [...snip...]
}
```

Here we can see that a class has a collection of method declarations. It is represented in two ways, firstly as an instance of `TreeNodeCollection<T>` and secondly as a `TreeNodeEnumerable<T>`. Both can be iterated over using `foreach`, or LINQ queries. The major difference is that `TreeNodeCollection<T>` implements `IList<T>`, while `TreeNodeEnumerable<T>` only implements `IEnumerable<T>`. This means that the `MethodDeclarationsEnumerable` property is more lightweight that the `MethodDeclarations` property, but at the cost of flexibility.

The `MethodDeclarations` property creates a new instance of `TreeNodeCollection<T>` each time it's accessed. The child nodes are iterated up-front, and passed to the collection as an array. The collection provides all the features of an implementation of `IList<T>` (although it is a read-only list), so it can be iterated sequentially or randomly. It also provides a number of optimised LINQ queries, such as `Any`, `Reverse` and `ToList`.

The `MethodDeclarationsEnumerable` property, however, simply creates an instance of `TreeNodeEnumerable<T>` (again, each time it's accessed), but the nodes are not collected until the enumerable is enumerated - it is lazily evaluated, and does not store the whole list. This makes it more efficient than `TreeNodeCollection<T>`.

 >  You should favour the enumerable version of a property unless you require the extra features that `IList<T>` gives you.
 >
 {type="tip"}

## Navigating up the tree

ReSharper also provides a strongly typed means of navigating *up* the tree, which is preferable to manually walking up the chain of `Parent` properties. Generally speaking, if a node exposes strongly typed accessors, there will be a complementary navigator class.

For example, the `IXmlTag` node exposes a `Header` property of type `IXmlTagHeader`, providing us strongly typed navigation *down* the tree (from a node representing a complete xml tag to the node representing the opening part of the tag). The `XmlTagNavigator` static class gives us the opposite - strongly typed navigation *up* to an `IXmlTag` from various child nodes.

```csharp
public static class XmlTagNavigator
{
  public static IXmlTag GetByTag(IXmlTag tag) { /* … */ }
  public static IXmlTag GetByTagHeader(IXmlTagHeader header) { /* … */ }
  public static IXmlTag GetByTagFooter(IXmlTagFooter footer) { /* … */ }
  public static IXmlTag GetByAttribute(IXmlAttribute attribute) { /* … */ }
}
```

The classes follow a simple naming pattern. The navigator class is named after the node you're trying to navigate *to*, and the name ends in the word `Navigator`, e.g. `XmlTagNavigator` is trying to navigate to an instance of `IXmlTag`. The methods all being with `GetBy` and finish with the type of the node that should be passed in. So, `XmlTagNavigator.GetByAttribute` is trying to navigate to an `IXmlTag`, from an `IXmlAttribute`.

 >  Navigating up the tree is not as simple as casting `Parent` to the appropriate type. The navigator methods have specific knowledge of parent/child relationships, and can walk several nodes up the hierarchy to find the appropriate node. For example, C#'s `ClassDeclarationNavigator` can navigate to the class declaration node from an `IAttribute` node, skipping the attribute list, attribute section and section list nodes before reaching the class declaration node.
>
> Since ReSharper knows these relationships, it is recommended that you use navigator classes where possible, instead of walking the `Parent` properties manually.
 >
 {type="warning"}

It should also be pointed out that you can use the navigator classes in combination. For example, to get to a class declaration from an `if` statement in a C# file:

```csharp
public IClassDeclaration GetByIfStatement(IIfStatement ifStatement)
{
  var methodBody = BlockNavigator.GetByStatement(ifStatement);
  var methodDeclaration = MethodDeclaratioNavigator.GetByBody(methodBody);
  return ClassDeclarationNavigator.GetByMethodDeclaration(methodDeclaration);
}
```

It isn't necessary to check for null before passing a node into the navigator classes - they explicitly allow null to be passed in, and will return null if it can't find the appropriate parent.