# Control Flow Graphs

<!-- toc -->

A [control flow graph](http://en.wikipedia.org/wiki/Control_flow_graph) is a data structure that can represent all paths through a block of code. For example, in a simple function with no branches or return statements, a single path can be traced through all statements in the function. Given an `if` condition in the code, the control flow graph now contains more than one path, with multiple paths for the `true` and `false` statements of the `if` expression.

A graph is represented with the `IControlFlowGraph` interface. Each node in the graph is an `IControlFlowElement` and the connecting edges of the graph are represented by `IControlFlowEdge`.

ReSharper uses control flow graphs to perform control flow analysis, including looking for redundant, unreachable code, and performing value analysis, locating potential null references or known unnecessary null checks.

### Edges

The edge is the simplest interface:

```cs
public interface IControlFlowEdge
{
  ControlFlowEdgeType Type { get; set; }
  IControlFlowElement Source { get; }
  IControlFlowElement Target { get; }
  int Id { get; }
  ITreeNode GetSourceElement();
}
```

It connects two elements together, and provides a type, which declares how the `Source` and `Target` elements are connected:

* `NEXT` - the target element is simply the next element.
* `GOTO` - a `goto` statement that transfers control from the source to the target elements.
* `THROW` - when an exception is thrown at the source element, the control flow transfers to the target element.
* `RETURN` - a return statement transfers control from the source to the target elements.
* `PHANTOM_NEXT` - an edge that isn't part of the physical control flow, used for "what-if" analysis. Discussed below.

It also contains a simple integer ID, and a reference to the `ITreeNode` that represents the `Source` element's tree node (useful for navigating to a function's exits, for example).

### Elements

An edge connects two elements together. An element is associated to an `ITreeNode`, usually mapping to a statement, expression or block. The element interface:

```cs
public interface IControlFlowElement
{
  IControlFlowElement Parent { get; }
  ITreeNode SourceElement { get; }
  bool IsReachable { get; }
  Iist<IControlFlowElement> Children { get; }
  List<IControlFlowEdge> Entries { get; }
  List<IControlFlowEdge> Exits { get; }
  IControlFlowEdge PhantomExit { get; }
  int Id { get; }
}
```

## Phantom edges

## Utility methods

The `ControlFlowBuilder` class provides several useful static utility methods:

* **`GetGraph`** - return the graph for the given `ITreeNode`. The tree node should be a node that can own a control flow graph, such as a method declaration. The `buildExpressions` parameter controls whether expressions are built for the control flow graph or not. Returns `null` if a graph can't be created on the node.
* **`GetContainingGraph`** - returns the graph that contains the given `ITreeNode`. This will try and create a graph at the given node, and if that fails, walk the parent chain of tree nodes until it finds a node that can create a graph, or it finds the root of the tree, in which case it returns `null`.
* **`GetContainingGraphOwner`** - checks the given node and walks up the parent chain to find the node that can create a graph.

The `ControlFlowElementExtensions` class provides extension methods:

* **`Contains(this IControlFlowElement element, IControlFlowElement candidate)`** - returns true if the `candidate` flow element is a child of the `element` parameter.
* **`Descendants`** - recursively returns a list of all of the children of the given flow element.
* **`ReachableExits`** - returns a list of the exits of all reachable leaf descendants of the given element.

## Language support

ReSharper supports control flow graphs on four languages, out of the box, but the mechanism is, of course, extensible.

### C#

ReSharper can build a control flow graph for the following C# tree nodes:

* `ICSharpFunctionDeclaration` - methods, constructors, finalisers, operators and properties.
* `IExpressionBodyOwnerDeclaration` - C# 6 constructs that support expression bodies, including properties, property indexers, methods, operators.
* `IAnonymousFunctionExpression` - lambda expressions
* `IQueryParameterPlatform` - LINQ expression queries (`from ... in ...`)
* `IClassDeclaration` and `IStructDeclaration`

### Visual Basic

Visual Basic support is similar to C#, but limited to:

* `IVBFunctionDeclaration` - methods, constructors, finalisers, operators and properties.
* `ILambdaExpression` - lambda expressions.
* `IQueryParameterPlatform` - LINQ expression queries.

### JavaScript and TypeScript

JavaScript supports control flow graphs creates on the following tree nodes:


* `IJsFunctionLike` - functions and accessors.
* `IJavaScriptFileSection` - global code defined inline in the main code file.

TypeScript extends JavaScript's support with:

* `ITsFunctionLike` - TypeScript functions, including constructors, function expressions and statements, member accessors and functions and object property functions.
* `ITsExpressionLambdaExpression` - lambda expressions.

## Implementing control flow graph support

`FixUp` + `FixUpRecursive`
