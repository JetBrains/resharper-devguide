[//]: # (title: Navigate to a Particular Tree Node)

**What you should know beforehand:**
* [PSI](NavigateCode.md#psi-basics)

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [NodeUnderCaretDetector.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/NodeUnderCaretDetector.cs)

In case you have already obtained your navigation target, e.g., an `ITreeNode` instance, you can use its `NavigateToTreeNode()` method to navigate to it:

```csharp
ITreeNode node = ... ;
node.NavigateToTreeNode(true);
```

Other PSI syntax tree classes may also provide methods for navigation, e.g. `IDeclaredElement`:

```csharp
IDeclaredElement element = ... ;
element.Navigate(true);
```
