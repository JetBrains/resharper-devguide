[//]: # (title: Bulk Actions)

 >  This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.
 >
 {type="warning"}

A Bulk Action is a Quick-Fix or a Context Action that can be applied to an element in the syntax tree, a single file, or all files in a folder, project or solution. This [fix in scope](http://www.jetbrains.com/resharper/webhelp/Code_Analysis__Fix_in_Scope.html) mechanism is displayed as a menu item on the `Alt+Enter` menu that can be selected to affect the single item, or expanded to show the per-file, folder, project or solution scope:

![Alt+Enter menu showing fix in scope context action](fix_in_scope.png)

This mechanic is available for both quick-fix actions as well as context actions that are part of Code Cleanup.

In the case of a context action (such as ReSharper’s `VarToTypeAction`, which converts the C# `var` keyword to an explicit type), the implementation of additional items happens in the `CreateBulbItems()` method. A context action can, of course, create all the required bulb items directly, but it can also use elements from the `JetBrains.ReSharper.Intentions.Bulk` namespace to simplify the process.

In the case of `VarToTypeAction`, ReSharper uses the `BulkCodeCleanupContextActionBuilder` class to construct all the items as follows:

```csharp
var cleanupProfile = BulkCodeCleanupActionBuilderBase.CreateProfile(profile =>
  {
    profile.SetSetting(ReplaceByVar.USE_VAR_DESCRIPTOR
  }, new ReplaceByVar.Options()
  {
    BehavourStyle = ReplaceByVar.BehavourStyle.CAN_CHANGE_TO_EXPLICIT,
    ForeachVariableStyle = ReplaceByVar.ForeachVariableStyle.ALWAYS_EXPLICIT,
    LocalVariableStyle = ReplaceByVar.LocalVariableStyle.ALWAYS_EXPLICIT
  }));

var projectFile = myProvider.SourceFile.ToProjectFile();
Assertion.AssertNotNull(projectFile, "projectFile must not be null");

var builder = BulkCodeCleanupContextActionBuilder.CreateByPsiLanguage<CSharpLanguage>(cleanupProfile,
  "Use explicit type everywhere", projectFile.GetSolution(), this);
return builder.CreateBulkActions(projectFile, IntentionsAnchors.ContextActionsAnchor,
  IntentionsAnchors.ContextActionsAnchorPosition);
```

The exact mechanic of creating bulk context actions can be found in `BulkIntentionsBuilderEx.CreateBulkActions()`. It is a rather simple process that creates actions for the current caret position, then (if applicable) the current file, project folder, project or the whole solution.

Quick-fixes have a similar workflow - create a builder and in your call `CreateBulbItems` method, return `builder.CreateBulkActions()`. The builder to use for quick-fixes is called `BulkQuickFixWithCommonTransactionBuilder`, and can be used in a couple of ways. As of ReSharper 8.2, it is internally used for quick-fixes such as `OptimizeImportsFix`, which removes all redundant C# "using" statements in a file, and optionally, folder, project and solution. In other words, the default implementation of the quick fix operates on the whole file, rather than the current text caret position.

As such, it has a slightly awkward API, and requires a little extra work to support an action at the caret position as well as file, folder, project and solution scope. The constructor has two overloads. The first takes in a `QuickFixBase` that represents the "in file" scope, as well as a delegate to operate on files in folder, project and solution scope, and a predicate to see which file in scope it should apply to (e.g. just C# files). The second overload takes two `QuickFixBase` instances. One is for the caret position, and the second is for the "in file" scope. It also requires the delegate and predicate.

The awkward part of this is that to support both "at caret" and "in file" scope, you need to implement two classes that derive from `QuickFixBase`. Fortunately, this is mostly boilerplate and can be implemented using the same delegate passed in for folder, project and solution scope. You need just one class that looks something like this:

```csharp
public class BulkQuickFixInFileWithCommonPsiTransaction : QuickFixBase
{
  private readonly IProjectFile myProjectFile;
  private readonly Action<IDocument, IPsiSourceFile, IProgressIndicator> myPsiTransactionAction;

  public BulkQuickFixInFileWithCommonPsiTransaction (IProjectFile projectFile, string actionText,
    Action<IDocument, IPsiSourceFile, IProgressIndicator> psiTransactionAction)
  {
    myProjectFile = projectFile;
    myPsiTransactionAction = psiTransactionAction;
    Text = actionText + " in file";
  }

  public override bool IsAvailable(IUserDataHolder cache)
  {
    return true;
  }

  public override string Text { get; private set; }

  public override Action<ITextControl> ExecutePsiTransaction(ISolution solution, IProgressIndicator progress)
  {
    var documentManager = solution.GetComponent<DocumentManager>();
    var document = documentManager.GetOrCreateDocument(myProjectFile);
    var psiSourceFile = projectFile.ToSourceFile();
    if (psiSourceFile == null)
      return;

    myPsiTransactionAction(document, psiSourceFile, progress);
    return null;
  }
}
```

So, if your quick-fix operates on just the caret position, you should do something like this:

* Implement the quick-fix. The `ExecutePsiTransaction()` method just fixes the issue at the text caret position
* Prepare an action that processes a single file with your chosen logic. Here’s an example:

    ```csharp
    var solution = projectFile.GetSolution();
    var psiFiles = solution.GetComponent<IPsiFiles>();
    Action<IDocument, IPsiSourceFile, IProgressIndicator> processFileAction =
      (document, psiSourceFile, indicator) =>
    {
      var csharpFiles = psiFiles.GetPsiFiles<CSharpLanguage>(psiSourceFile).OfType<ICSharpFile>();
      foreach (var csharpFile in csharpFiles)
      {
        UsingUtil.RemoveUnusedImports(document, csharpFile);
      }
    };
    ```

* Create an instance of the `BulkQuickFixInFileWithCommonPsiTransaction` class defined above (make sure to add this class to your project), passing in an instance of the `processFileAction` delegate

    ```csharp
    var inFileFix = new BulkQuickFixInFileWithCommonPsiTransaction(projectFile, RemoveUnusedDirectivesString, processFileAction);
    ```

* Create a `BulkQuickFixWithCommonTransactionBuilder`, which also happens to accept a predicate that you can use to filter out files you do not want to process:

    ```csharp
    var acceptProjectFilePredicate = BulkItentionsBuilderEx.CreateAcceptFilePredicateByPsiLanaguage<CSharpLanguage>(solution);
    var builder = new BulkQuickFixWithCommonTransactionBuilder(this, inFileFix, solution, RemoveUnusedDirectivesString, processFileAction, acceptProjectFilePredicate);
    ```

* Finally, use the builder to create all the actions in bulk:

    ```csharp
    return builder.CreateBulkActions(projectFile, IntentionsAnchors.QuickFixesAnchor, IntentionsAnchors.QuickFixesAnchorPosition);
    ```

And if your quick-fix operates on the whole file by default, and doesn't operate only at the issue at the caret position (e.g. remove all redundant using statements):

* Implement the quick-fix. The `ExecutePsiTransaction()` method fixes the issue across the whole file
* Prepare an action that processes a single file with your chosen logic. The example here is the same as above:

    ```csharp
    var solution = projectFile.GetSolution();
    var psiFiles = solution.GetComponent<IPsiFiles>();
    Action<IDocument, IPsiSourceFile, IProgressIndicator> processFileAction = (document, psiSourceFile, indicator) =>
    {
      var csharpFiles = psiFiles.GetPsiFiles<CSharpLanguage>(psiSourceFile).OfType<ICSharpFile>();
      foreach (var csharpFile in csharpFiles)
      {
        UsingUtil.RemoveUnusedImports(document, csharpFile);
      }
    };
    ```

* Create a `BulkQuickFixWithCommonTransactionBuilder`, together with a predicate to indicate which files you want to process. Note that we call a different constructor here, and pass in `this` as the quick fix to operate by default. This overload treats this quick-fix as the "in file" scope quick fix, and assumes the "at caret position" quick-fix isn't required:

    ```csharp
    var acceptProjectFilePredicate = BulkItentionsBuilderEx.CreateAcceptFilePredicateByPsiLanaguage<CSharpLanguage>(solution);
    var builder = new BulkQuickFixWithCommonTransactionBuilder(this, solution, RemoveUnusedDirectivesString, processFileAction, acceptProjectFilePredicate);
    ```

* Finally, use the builder to create all the actions in bulk:

```csharp
return builder.CreateBulkActions(projectFile, IntentionsAnchors.QuickFixesAnchor, IntentionsAnchors.QuickFixesAnchorPosition);
```