---
---

# Obtain Components in Runtime
**Examples:**
* [SampleSolutionComponent.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/SolutionComponent/SampleSolutionComponent.cs)
* [SolutionStateTracker.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/SolutionStateTracker/SolutionStateTracker.cs)
 
ReSharper uses the loosely coupled design. ReSharper's [component model](/Platform/ComponentModel.md) provides the ability to live load and unload new components, during runtime. 

Typically, you as a plugin developer should use only shell and solution components:
* **Shell components** have the same lifetime as ReSharper and are marked with the `ShellComponent` attribute.
* **Solution components** have the same lifetime with a solution, thus, they are created once a solution is loaded and terminated once the solution is closed. Solution components should be marked with the `SolutionComponent` attribute.

So, how could you obtain your own or ReSharper components in runtime? Let's take a look at the most common scenarios.

## Constructor injection
The easiest way is to use a constructor injection. All you need is to specify the required component as a constructor argument. ReSharper will care about all the dependencies on its own.

For example, you have a solution component `MyClass` that should run some action upon creation. To make this happen, it must obtain an `ActionManager` instance.

```
    [SolutionComponent]
    public class MyClass
    {
        public MyClass(Lifetime lifetime, IActionManager actionManager)
        {
            actionManager.ExecuteAction<SomeAction>());
        }
    }
```
Here `SomeAction` is action's name.

## Current context
Sometimes, you can get components from the `IDataContext` objects. This is true, for example, for [actions](/HowTo/CreateMenuItemsUsingActions.md):
```
protected override void RunAction(IDataContext context, DelegateExecute nextExecute)
{
    var actionManager = context.GetComponent<IActionManger>();
    ...
 }
```
## Shell and solution component
Another way is to get shell components via the `Shell` component - the root API point. This can be helpful, for example, if you want to have some "global" settings and access them from anywhere inside your plugin. To obtain the `Shell` instance, you must use its `Instance` property.
```
[ShellComponent]
public class MySettings
{
    public bool AutoRefresh { get; set; }
 
    public MySettings()
    {
        AutoRefresh = true;
    }
 
    public static MySettings Instance
    {
        get { return Shell.Instance.GetComponent<MySettings>(); }
    }
}
```
Thus, you can simply access them from any point of your plugin:
```
if (MySettings.Instance.AutoRefresh) ...
``` 
Same is true for solution components. The main difference is that you should have an object that implements the `ISolution` interface (the one that refers to the solution currently opened in Visual Studio):
```
public void ObtainSomeSolutionComponent(ISolution solution)
{
    var component = solution.GetComponent<SomeSolutionComponent>();
}
 
...
 
[SolutionComponent]
public class SomeSolutionComponent {}
```
