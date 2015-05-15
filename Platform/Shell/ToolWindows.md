---
---

# Tool Windows

One of ReSharper's core abilities is to present various types of tool windows. Broadly speaking, there are two varieties of tool window: one where its content structure is predefined, and one where you decide what the content of the window is.

## General-purpose tool window

The general-purpose tool window typically consists of the following:

* **Descriptor** - an empty metadata-carrying class for describing the tool window particulars.
* **Registrar** - a component that is used to register the tool window with the tool window manager.
* **Action** - the action which, when fired, brings up the tool window.
* The window class, if you need one. Alternatively, you can create UI from existing building blocks.

### Descriptor

The descriptor is simply an empty class that inherits from `ToolWindowDescriptor` and is decorated with the `ToolWindowDescriptor` attribute. All of the configuration happens in the attribute. Here is an example:

```csharp
[ToolWindowDescriptor(
  ProductNeutralId="KeyboardHelperWindow",
  Text="Keyboard Helper",
  Guid="4338B272-3CD7-4DA3-98D2-4AD0025FAB86",
  Type = ToolWindowType.SingleInstance,
  VisibilityPersistenceScope = ToolWindowVisibilityPersistenceScope.Global,
  Icon = typeof(Resources.ServicesThemedIcons.Keyboard),
  InitialDocking = ToolWindowInitialDocking.Right
  )]
public class KeyboardHelperDescriptor : ToolWindowDescriptor
{
}
```

Let us briefly go over the information that is provided in the above:

* **ProductNeutralId** - this is the tool window's identifier. Note that, unlike with Actions, this identifier isn't used for instantiation.
* **Text** - the text that appears in the title of the tool window.
* **Guid** - a unique identifier for this window. Use the `nguid` live template to make one.
* **Type** - determines whether the tool window is single- or multi-instance.
* **VisibilityPersistenceScope** - controls how the tool window's visibility is recorded. The tool window's visibility can be associated with a particular solution, or can be global, i.e., apply to all solutions.
* **Icon** - the type of a resource identifier for the icon that appears in the tool window.
* **InitialDocking** specifies the initial part of the window in which the tool window is docked when shown.

### Action

To create an action that opens up a tool window, simply create a class that inherits from `ActivateToolWindowHandle<TDescriptorType>`, parameterise it with your descriptor type (as seen above) and decorate it with the `ActionHandler` attribute:

```csharp
[ActionHandler("MyPlugin.ShowKeyboardHelper")]
public class KeyboardHelperAction : ActivateToolWindowActionHandler<KeyboardHelperDescriptor>
{
}
```

### Registrar

The registrar is a component (it can be shell or solution) that is used to register the tool window class with the tool window manager. Actual UI creation happens here. The UI can be created in one of the two ways.

The first option is to simply use a `UserControl` (it can be either WinForms or WPF):

```csharp
[ShellComponent]
public class KeyboardHelperRegistrar
{
  private readonly Lifetime lifetime;
  private readonly ToolWindowClass toolWindowClass;

  public KeyboardHelperRegistrar(Lifetime lifetime, ToolWindowManager toolWindowManager, IVsUIShell shell,
    IActionManager actionManager, IShortcutManager shortcutManager, KeyboardHelperDescriptor descriptor)
  {
    this.lifetime = lifetime;

    toolWindowClass = toolWindowManager.Classes[descriptor];
    toolWindowClass.RegisterEmptyContent(lifetime, lt => {
      var window = new KeyboardHelperWindow(shell);
      return window.BindToLifetime(lt);
    });
  }
}
```

Another alternative it to construct the UI _in situ_, without creating a separate control class. To do this, you can use ReSharper-provided controls (such as `RichTextBox`) and subsequently use the same `BindToLifetime()` method in much the same fashion.

