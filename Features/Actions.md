---
title: Actions and Menu Items
---

> **WARNING** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

While the bulk of ReSharper is concerned with presenting its own types of UI (popups, dialog windows, etc.), certain features can be implemented using traditional menu items that would be familiar to anyone who has done Visual Studio plugin development. A menu item subsequently triggers an _action_, i.e. a unit of work associated with this menu item.

## When to create Actions

There are essentially two use cases for creating Actions.

The first use case is when you want to expose a particular ReSharper feature, such as e.g., a Context Action, as a Visual Studio command that can be bound to a shortcut. The relevance of this depends on the feature.

The second use case would be a situation where you want your action to do things unrelated to the ReSharper infrastructure. Examples of such actions would be an action to show an About dialog box, to open up a Help File for your plugin, or fire up some other process.

Both of these use cases are accommodated by the Action subsystem.

## Making actions

In order to add your own menu items to the infrastructure, you need to do the following things:

* Create an `Actions.xml` file which will contain definitions of actions and the places where to insert them.
* Ensure the `Actions.xml` file is embedded as a resource in your assembly
* Ensure that the assembly has an ActionsXml attribute pointing to the above file.
* Define action handlers for the actions.

Let's take a look at these steps in turn.

## Actions.xml

`Actions.xml` is an embedded resource you can keep in your project. ReSharper is able to locate the resource due to the `ActionsXml` attribute with which the assembly is decorated. For example:

```csharp
[assembly: ActionsXml("JetBrains.ReSharper.PowerToys.MenuItem.Actions.xml")]
```

> **NOTE**: This attribute is no longer required in ReSharper 8. ReSharper will instead look for all embedded resources in an assembly to see if it matches the name `Actions.xml`

The XML file itself is fairly simple. At the root level, it uses the `actions` element. Within it, there are several different types of content.

One mandatory type of content is action elements, which indicate the ID of the action concerned, the text that gets displayed, and (optionally) an image. The image can be, e.g., a 16x16 pixel GIF stored as an embedded resource.

```xml
<action id="AddMenuItem.ShowCurrentSolution" text="Show Current Solution"
        image="JetBrains.ReSharper.PowerToys.MenuItem.testImage.gif" />
```

This is the way actions are defined, but how are they inserted into existing menus? This is what the insert element is for. In this element, you specify the Group ID where the items are inserted, as well as the position they are inserted in (e.g., "last").

When specifying the group ID, you can specify

* A ReSharper menu (e.g., "ReSharper.Navigate")
* A Visual Studio menu (e.g., "VS#Solution")

For Visual Studio, the part following VS# is the path in the command bar structure of VS itself. You can also use the menu CommandID for this, in the form "VS#(GUID:ID)".

Consequently, to insert a separator and the ShowCurrentLineNumber action into the VS code window, you would add the following:

```xml
<insert group-id="VS#Code Window" position="last">
  <separator />
  <action-ref id="AddMenuItem.ShowCurrentLineNumber" />
</insert>
```

## Action Handlers

An action handler is the piece of code that handles an action. It is simply a class decorated by the `IActionHandler` interface and implementing the `IActionHandler` interface.

Now, you're probably wondering - how is the action handler bound to an action? Well, there are two ways to do this. One is to name the class file to match the action, such that an action of `AddMenuItem.ShowCurrentSolution` matches the class name of `AddMenuItem_ShowCurrentSolutionAction`. As you may have guessed, underscores get 'turned' into dots, and suffixes such as `Action` or `ActionHandler` are removed.

Concerning the `IActionHandler` interface, it requires that you implement two methods.

The first - `Update()` - returns a boolean value indicating whether the menu item is enabled or disabled. For example, if your action needs a solution to work on, you could specify the following Update() method:

```csharp
public bool Update(IDataContext context, ActionPresentation presentation, DelegateUpdate nextUpdate)
{
  // fetch active solution from context
  ISolution solution = context.GetData(JetBrains.ProjectModel.DataContext.DataConstants.SOLUTION);

  // enable this action if there is an active solution, disable otherwise
  return solution != null;
}
```

The second method is called `Execute()`. This is the method where the action is actually executed.

## Redefining Action Handlers

If you wish to override existing actions, you can create a custom shell component that finds an existing action and adds a different handler for it. For example, here is a custom action handler override for the `TypeHierarchy.ClassHierarchy` action:

```csharp
[ShellComponent]
public class CustomActionHandlerManager
{
  public CustomActionHandlerManager(Lifetime lifetime, IActionManager actionManager)
  {
    var classHierarchy = actionManager.GetUpdatableAction("TypeHierarchy.ClassHierarchy");
    if (classHierarchy != null)
      classHierarchy.AddHandler(lifetime, new WholeMembersHierarchyActions());
  }
}
```

