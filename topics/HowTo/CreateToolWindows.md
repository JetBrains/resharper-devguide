[//]: # (title: Create Tool Windows)

**What you should know beforehand:**
* [Component model](ObtainComponentsInRuntime.md)
* [Lifetime](WorkWithLifetime.md)

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [SampleToolWindowDescriptor.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/ToolWindow/SampleToolWindowDescriptor.cs) - descriptor
* [SampleToolWindow.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/ToolWindow/SampleToolWindow.cs) - registrar
* [OpenToolWindowAction.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Actions/OpenToolWindowAction.cs) - action that shows the window

ReSharper SDK provides you an ability to create your own tool windows. There is a special ReSharper `ToolWindowManager` component that manages all tool windows. To create a tool window means to register it in `ToolWindowManager` and provide the manager with the window description.

So, here is what you need to create a simple tool window (the minimal set):
* Descriptor
* Registrar

[Learn more about tool windows](ToolWindows.md).

## Tool Window Descriptor
As the name suggests it describes your future window: type, ID, caption, icon, and other parameters. Note that it does not provide the window content, i.e. UI controls.

```csharp
[ToolWindowDescriptor(
        ProductNeutralId = "MyToolWindow",
        Text = "My Plugin",
        Icon = typeof(JetBrains.Ide.Resources.IdeThemedIcons.TextDocument),
        Type = ToolWindowType.MultiInstance,
        VisibilityPersistenceScope = ToolWindowVisibilityPersistenceScope.Global,
        InitialDocking = ToolWindowInitialDocking.Right            
    )
]
public class MyToolWindowDescriptor : ToolWindowDescriptor
{
    public MyToolWindowDescriptor(IApplicationHost host) : base(host)
    {
    }
}
```

## Tool Window Registrar
This is a component that registers tool window in the tool window manager.

```csharp
public class MyToolWindow
{
    private readonly TabbedToolWindowClass _toolWindowClass;
    private readonly ToolWindowInstance _toolWindowInstance;
 
    public MyToolWindow(Lifetime lifetime, ToolWindowManager toolWindowManager,
        MyToolWindowDescriptor toolWindowDescriptor, IUIApplication uiApplication)
    {
        _toolWindowClass = toolWindowManager.Classes[toolWindowDescriptor] as TabbedToolWindowClass;
        if (_toolWindowClass == null)
            throw new ApplicationException("ToolWindowClass");
 
        _toolWindowInstance = _toolWindowClass.RegisterInstance(lifetime, "My Tool Window", null,
            (lt, twi) =>
            {
                var label = new RichTextLabel(uiApplication)
                {                        
                    Dock = DockStyle.Fill
                };
 
                label.RichTextBlock.Add(new RichText("Hello World!", new TextStyle(FontStyle.Bold)));
                label.RichTextBlock.Parameters = new RichTextBlockParameters(8, ContentAlignment.MiddleCenter);
 
                return new EitherControl(lt, label);
            });
    }

    public void Show()
    {
        _toolWindowInstance.Show();
    }
}
```

## Notes
* First you should get an instance of the `ToolWindowClass` or `TabbedToolWindowClass` (for window that uses tabs to show other tool windows). Then you can use this object to register a tool window instance (`_toolWindowInstance` in our example).
* Note that if you want to run some routines on window close, you can make it by using  the `ToolWindowClass.QueryCloseInstances` signal. Don't forget to set the tool window instance's `QueryClose` property to true.
* A delegate passed to the `RegisterInstance` method must return an instance of the `EitherControl` type. This is the place where you create tool window UI.
* To show the window, you must obtain all the required components, e.g., if you use an action:

```csharp
[Action("ActionOpenMyToolWindow", "Open a sample tool window", Id = 543211)]
public class ActionOpenMyToolWindow : SampleAction, IInsertLast<MainMenuFeaturesGroup>
{
    protected override void RunAction(IDataContext context, DelegateExecute nextExecute)
    {
        var lifetime = context.GetComponent<Lifetime>();
        var toolWindowManager = context.GetComponent<ToolWindowManager>();
        var toolWindowDescriptor = context.GetComponent<MyToolWindowDescriptor>();
        var environment = context.GetComponent<IUIApplication>();
        var toolWindow = new MyToolWindow(lifetime, toolWindowManager, toolWindowDescriptor, environment);
        toolWindow.Show();
    }
}
```
