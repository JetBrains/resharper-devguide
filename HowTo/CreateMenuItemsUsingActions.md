---
---

# Create Menu Items Using Actions

**Examples ([?](HowTo.md#sample-solution)):**
* [SampleAction.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Actions/SampleAction.cs) 
 
If you want to expose your feature in a menu or a toolbar, you should use actions. Action is a unit of work (implemented as a special class) that is associated with a particular menu or toolbar item. Note that you can assign a Visual Studio shortcut to an action. [Learn more about actions](/Features/Actions.md).
 
Let's create a simple action that shows a message box being triggered.
``` 
    public abstract class SampleAction : IExecutableAction
    {
        public bool Update(IDataContext context, ActionPresentation presentation, DelegateUpdate nextUpdate)
        {
            return true;    // function result indicates whether the menu item is enabled or disabled
        }
 
        public void Execute(IDataContext context, DelegateExecute nextExecute)
        {
            RunAction(context, nextExecute);
        }
 
        protected abstract void RunAction(IDataContext context, DelegateExecute nextExecute);        
    }
 
    [Action("ActionShowMessageBox", "Show message box", Id = 543210)]
    public class ActionShowMessageBox : SampleAction, IInsertLast<MainMenuFeaturesGroup>
    {
        protected override void RunAction(IDataContext context, DelegateExecute nextExecute)
        {
            var solution = context.GetData(JetBrains.ProjectModel.DataContext.ProjectModelDataConstants.SOLUTION);
            MessageBox.ShowInfo(solution?.SolutionFile != null
                ? $"{solution.SolutionFile?.Name} solution is opened"
                : "No solution is opened");
        }
    }
```
 
## Notes
* Actions must implement `IExecutableAction`.
* `Update` returns `bool` value that indicates whether the menu item is enabled or disabled.
* `Execute` runs the action.
* Actions are marked with the special `Action` attribute, which defines action name, text that is shown in a menu and action `Id`. Note that `Id` must be unique!
* Use the `IDataContext` object passed to the action to obtain any context data, e.g. [project model](/HowTo/NavigateCode/NavigateCode.md#project-model-basics) data.
