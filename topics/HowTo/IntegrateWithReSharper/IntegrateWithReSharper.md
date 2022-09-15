[//]: # (title: Integrate with ReSharper)

ReSharper uses loosely coupled principle. To develop a ReSharper plugin means to declare and implement a particular integration point. ReSharper will care of the rest: the extension will be injected into ReSharper.
 
How does this actually look like? Typically, you: 
* Mark a class with one of the special ReSharper attributes. For example, the `[Action]` attribute tells ReSharper that this class describes an action (one of the integration points) and will register it in the action manager.
* Work with ReSharper components that provide the functionality you need in your plugin. To obtain them, you specify the required components in the class constructor. ReSharper provides you the components using dependency injection. See an [example](ObtainComponentsInRuntime.md#constructor-injection). 
 
Below you will find the list of most common ReSharper integration points.
 
## Daemons and daemon stages

![daemons and daemon stages](daemons.png)

It's all about as-you-type code analysis. For example, ReSharper displays "squiggly" warnings in your code.

Examples in this **How To**:
* [Analyze Code on the Fly](AnalyzeCodeOnTheFly.md)
 
## Actions 

![actions](actions.png)

These are menu items, and toolbar items. You can expand the menu and the toolbar with your own items.

Examples in this **How To**:
* [Create Menu Items Using Actions](CreateMenuItemsUsingActions.md)
 
## Quick-fixes and context actions

![quick-fixes and context actions](quick-fixes.png)

These are context-dependent actions that ReSharper suggests by hitting <kbd>Alt</kbd>+<kbd>Enter</kbd>. ReSharper SDK allows you creating your own quick-fixes and context actions.

Examples in this **How To**:
* [Create a Context Action](CreateContextAction.md)
* [Create a Quick-Fix](CreateQuickFix.md)
 
## Code completion

![code completion](code-completion.png)

These are code completion menus that show suggestions based on user's typing.

Examples in this **How To**:
* no examples yet
 
## Reference providers

Apply references from an arbitrary abstract syntax tree node to another code element, such as supporting code completion and navigation from string literals to class or property names.
 
## Tool windows

![tool windows](tool-windows.png)

You can create your own tool windows by means of the ReSharper SDK.

Examples in this **How To**:
* [Create Tool Windows](CreateToolWindows.md)
 
## Generate menu
 
 ![generate menu](generate-menu.png)

These are items in the **Generate** menu that allow user to create a bulk of code quickly without really typing it. This menu is context-dependent. For example, ReSharper suggests to create a constructor if a class doesn't have it.

Examples in this **How To**:
* no examples yet
 
## Navigation

![navigation](IntegrateWithReSharper_navigation.png)

These are all ReSharper actions that relate to navigation, like **Go to Type**, **Go to Implementation**, and so on. You can extend this list with your own navigation actions.

Examples in this **How To**:
* [Add a Navigation Action to the **Navigate to** Menu](AddYourNavigationActionToNavigateToMenu.md)
 
## Options pages 

![options pages](options.png)

These are settings shown on the ReSharper's **Options** page. You can create your own settings and setting groups.

Examples in this **How To**:
* [Add Settings to ReShaper Options](AddSettingsToOptions.md)
 
## Refactorings

![refactorings](refactorings.png)

These are the code refactorings that ReSharper suggests depending on the context.

Examples in this **How To**:
* no examples yet
 
## Code cleanup 

ReSharper allows you to apply formatting and other code style preferences in a bulk mode to instantly eliminate code style violations in one or more files, in a project or in the entire solution.
 
Creating a new code formatting rule or editing an existing one doesn't require creating a compiled ReSharper plugin. That's why code cleanup is out of the scope of this **How To**. Learn how to work with code cleanup in [ReSharper documentation](https://www.jetbrains.com/help/resharper/Code_Cleanup__Index.html).
 
## Live templates

![live templates](live-templates.png)

Live templates are code fragments that can be quickly inserted into code. The code of the template can be a short expression, a complete construct, or even an entire class or method. For example, an `if...else` block, `foreach` block, `class` declaration, and so on.

A new live template can be created using the editor built in ReSharper and, therefore, doesn't require developing a compiled ReSharper plugin. That's why live templates are out of the scope of this **How To**. Learn how to create them in [Live Templates](LiveTemplates.md).
