---
---

# Generate Menu

So you’ve seen those fancy menu items in the _Generate_ menu (Alt+Insert) and you want to write your own? Okay, we’ll tell you how. But first, you have to realize that, really, those generator items are not specific to the _Generate_ menu _per se_. For example, the _Generate Missing Members_ item is mirrored as a Quick Fix. Still, we’ll be talking about generator items here.

## Provider

Right, let’s get started with the `GenerateProvider` - a provider class that determines which generator workflows are provided to the user. This may sound confusing so let’s go through it all one step at a time.

First, we create the generate provider body and appropriate attribute:

```csharp
[GenerateProvider]
public class GenerateDisposeItemProvider : IGenerateActionProvider
{
  public IEnumerable<IGenerateActionWorkflow> CreateWorkflow(IDataContext dataContext)
  {
    ...
  }
}
```

The above method `yield return`s all the necessary workflows, even if they are unavailable or not enabled. The classes it yields are expected to implement `IGenerateActionWorkflow`, but typically inherit from `StandardGenerateItemWorkflow`.

The workflow typically defines its own characteristic. However, we can pass an icon from the point where it is created:

```csharp
[GenerateProvider]
public class GenerateDisposeItemProvider : IGenerateActionProvider
{
  public IEnumerable<IGenerateActionWorkflow> CreateWorkflow(IDataContext dataContext)
  {
    var solution = dataContext.GetData(ProjectModel.DataConstants.SOLUTION);
    var iconManager = solution.GetComponent<PsiIconManager>();
    var icon = iconManager.GetImage(CLRDeclaredElementType.METHOD);
    yield return new GenerateDisposeActionWorkflow(icon);
  }
}
```

## Workflow

A workflow is simply a process defining how the action looks. Most of the `StandardGenerateActionWorkflow` that we’ll be using is actually a call to its base class that configures the various properties that subsequently affect the way the menu item is displayed.

```csharp
public class GenerateDisposeActionWorkflow : StandardGenerateActionWorkflow
{
  // Using named parameters for clarification
  public GenerateDisposeActionWorkflow(Image icon)
    : base(kind: "Dispose", icon: icon, menuText: "&Dispose",
           actionGroup: GenerateActionGroup.CLR_LANGUAGE,
           windowTitle: "Generate dispose",
           description: "Generate a Dispose() implementation which disposes selected fields.",
           actionId: "Generate.Dispose")
  {
  }

  public override double Order
  {
    get { return 100; }
  }
}
```

In the above declaration, we specify that we have a menu item with a _kind_ of “Dispose”. Note that this parameter is the glue which connects it to the generator element provider (more on this later), so it’s important to get it right.

Another thing you need to define is the order in which the item appears in the _Generate_ menu. It is recommended that you put in a suitably large value so that your menu item doesn’t get mixed with existing ones.

Now, all is good but the above won’t work. The problem is that we’ve defined our own item _kind_, which goes against ReSharper’s policy of only enabling items of a well-known kind. As a result, we also redefine the `StandardGenerateActionWorkflow.IsAvailable()` method, taking ReSharper’s own implementation and removing the _kind_ check:

```csharp
public override bool IsAvailable(IDataContext dataContext)
{
  var solution = dataContext.GetData(ProjectModel.DataConstants.SOLUTION);
  if (solution == null)
    return false;

  var generatorManager = GeneratorManager.GetInstance(solution);
  if (generatorManager == null)
    return false;

  var languageType = generatorManager.GetPsiLanguageFromContext(dataContext);
  if (languageType == null)
    return false;

  var generatorContextFactory = LanguageManager.Instance.TryGetService<IGeneratorContextFactory>(languageType);
  return generatorContextFactory != null;
}
```

