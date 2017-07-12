---
title: Get a Tree Node Under Caret
---

**What you should know beforehand:**
* [Component model](/HowTo/ObtainComponentsInRuntime.md)
* [Lifetime](/HowTo/WorkWithLifetime.md)
* [Project model](/HowTo/NavigateCode/NavigateCode.md#project-model-basics)
* [PSI](/HowTo/NavigateCode/NavigateCode.md#psi-basics)
* [IProperty](/HowTo/WorkWithIProperty.md)
* ShellLocks

**Examples ([?](/HowTo/HowTo.md#sample-solution)):**
* [NodeUnderCaretDetector.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/NodeUnderCaretDetector.cs)

Getting a tree node under the current caret position is a very typical task when implementing a navigation or any other tree-node-dependent feature. Actually, there are many ways of obtaining the tree node depending on the context. For example, when implementing [context navigation](/HowTo/NavigateCode/NavigateCode.md), you can use the `GetSelectedTreeNode` method of the `IDataContext` object. E.g.:

```csharp
// ...

public IEnumerable<ContextNavigation> CreateWorkflow(IDataContext dataContext)
{
    var node = dataContext.GetSelectedTreeNode<ITreeNode>();
    // ...
```

Nevertheless, if you need a more generic (context-independent) get-node-under-caret function implementation, see the example below:

```csharp
[SolutionComponent]
public class NodeUnderCaretDetector
{
    public ISolution Solution { get; }
    private readonly Lifetime _lifetime;
    private readonly DocumentManager _documentManager;
    private readonly ITextControlManager _textControlManager;
    private readonly IShellLocks _shellLocks;
    public IProperty<ITreeNode> NodeUnderCaret { get; set; }
 
    public NodeUnderCaretDetector(Lifetime lifetime, ISolution solution,
        DocumentManager documentManager,
        ITextControlManager textControlManager, IShellLocks shellLocks)
    {
        Solution = solution;
        _lifetime = lifetime;
        _documentManager = documentManager;
        _textControlManager = textControlManager;
        _shellLocks = shellLocks;
        NodeUnderCaret = new Property<ITreeNode>("NodeUnderCaretDetector.NodeUnderCaret");
 
        EventHandler caretMoved = (sender, args) =>
        {
            _shellLocks.QueueReadLock("NodeUnderCaretDetector.CaretMoved", Refresh);
        };
 
        lifetime.AddBracket(
            () => _textControlManager.Legacy.CaretMoved += caretMoved,
            () => _textControlManager.Legacy.CaretMoved -= caretMoved);
    }
 
    public void Refresh()
    {
        var node = GetTreeNodeUnderCaret();
        NodeUnderCaret.Value = node;
    }
 
    [CanBeNull]
    public ITreeNode GetTreeNodeUnderCaret()
    {
        var textControl = _textControlManager.LastFocusedTextControl.Value;
        if (textControl == null)
            return null;
 
        var projectFile = _documentManager.TryGetProjectFile(textControl.Document);
        if (projectFile == null)
            return null;
 
        var range = new TextRange(textControl.Caret.Offset());
 
        var psiSourceFile = projectFile.ToSourceFile().NotNull("File is null");
 
        var documentRange = range.CreateDocumentRange(projectFile);
        var file = psiSourceFile.GetPsiFile(psiSourceFile.PrimaryPsiLanguage, documentRange);
 
        var element = file?.FindNodeAt(documentRange);
 
        return element;
    }
}
```

## Notes
* The solution component `NodeUnderCaretDetector` has the public `IProperty` - `NodeUnderCaret`. You can use this property to obtain the tree node under the caret at any moment of time.
* The `NodeUnderCaret.Value` is updated each time the caret is moved. This is implemented by means of the legacy `CaretMoved` event handler.
* The `GetTreeNodeUnderCaret()` method performs all the work - it returns the `ITreeNode` element under the caret.
* The `textControl` instance (`ITextControl`) allows working with the currently opened text file as if you worked with a text editor. Here we obtain the current caret offset by calling `textControl.Caret.Offset()`.
* Note that, formally speaking, `textControl` knows nothing about the code inside itself - it simply renders the content it gets from an `IDocument` instance. 
* By knowing the document, we can obtain the [IProjectFile](NavigateCode.md#project-model-basics) (`_documentManager.TryGetProjectFile`) instance - it provides the API to work with the file's syntax tree.
* From the `IProjectFile`, we obtain, one after another [IPsiSourceFile](NavigateCode.md#project-model-basics) and [IFile](NavigateCode.md#psi-basics) (the main element of the PSI syntax tree).
* `IFile` provides the `FindNodeAt()` method that returns the node by the specified range.
* Note that to prevent conflicts with user input, we must obtain the read lock before calculating the node under the caret. This is done by means of the special method of the `IShellLocks` interface: `_shellLocks.QueueReadLock(...)`.
