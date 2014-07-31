# Refactoring

Refactorings are one of the most powerful features of ReSharper, but also the most complicated. A refactoring can be triggered from a variety of events (e.g., from a simple edit or deliberate invocation) and can result in all sorts of operations, some involving just local code, others involving the project (e.g., renaming a file) or changing multiple files in many projects.

At the most simple level, a refactoring consists of three things:

* The workflow
* The workflow provider
* The refactoring itself

Let's take a look at the way these components are defined.

## Workflow

The workflow, as its name suggests, is a class which orchestrates the workflow during which the user may (or may not) perform the refactoring. Most refactorings are _driven_, meaning that the user has to go through menus which present, e.g., conflicts that this refactoring would create.

In the simplest terms, a refactoring is a class which inherits from `DrivenRefactoringWorflow`. This class has quite a few members, most of which perform fairly obvious function.

Execution of the workflow happens in the `PreExecute`, `Execute` and `PostExecute` methods. All of these, as well as other methods such as `IsAvailable`, return a `bool` value indicating whether this step is a success. If any of these returns `false`, the whole transaction is rolled back and no refactorings take place.

## Workflow provider

The workflow provider is a class which provides workflows for ReSharper to consume. It is a class that

* Implements the `IRefactorignWorkflowProvider` interface
* Is decorated with `[RefactoringWorkflowProvider]` attribute

The class has a single method - `CreateWorkflow()` that returns an `IEnumerable` of one or more workflows that you may wish to provide. Here is a typical example:

```cs
[RefactoringWorkflowProvider]
public class MakeSingletonProvider : IRefactoringWorkflowProvider
{
  public IEnumerable<IRefactoringWorkflow> CreateWorkflow(IDataContext dataContext)
  {
    var solution = dataContext.GetData(JetBrains.ProjectModel.DataContext.DataConstants.SOLUTION);
    yield return new MakeSingletonWorkflow(solution, "MakeSingleton");
  }
}
```

The constructor parameter passed into the workflow is an action ID, which is also useful if you want to expose the refactoring as an ordinary action.

## The refactoring

One of the methods we forgot to mention in the workflow is the `CreateRefactoring()` method. This method is used to put together the refactoring worflow and the refactoring _driver_ - i.e., the element which drives the refactoring to completion.

Thus, the refactoring itself is typically a class (let's call it `C`) inheriting from `DrivenRefactoring<Workflow, Refactoring>`, where `Workflow` corresponds to the refactoring workflow, and `Refactoring` corresponds to a type of `RefactoringExecBase<Workflow, C>`.

```cs
public class MakeSingletonRefactoring
  : DrivenRefactoring<MakeSingletonWorkflow,
    RefactoringExecBase<MakeSingletonWorkflow,
    MakeSingletonRefactoring>>
{
  public MakeSingletonRefactoring(MakeSingletonWorkflow workflow, ISolution solution, IRefactoringDriver driver)
    : base(workflow, solution, driver)
  {
  }

  public override bool Execute(IProgressIndicator pi)
  {
    return true;
  }
}
```

The refactoring itself has the capability of dealing with cases where a particular refactoring is not supported on a ReSharper-supported language. For example, a particular C# refactoring might not be supported in VB, so usages after the change will not be updated. The number of such refactorings in ReSharper itself is really small, but it is likely you might need this functionality if your refactoring is, e.g., C#-only.

