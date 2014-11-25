# Quick-Fixes and Context Actions

> **Warning** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

One of the ways that ReSharper offers to correct or modify your code is by using a pop-up menu that shows up on the left-hand side of the screen:

![Showing the Context Action Alt+Enter menu](context_action.png)

This menu (typically triggered with the `Alt+Enter` shortcut) contains a set of menu items that plugin writers can provide. There are two separate ReSharper constructs that both can provide items to appear in this menu:

* _Context actions_ are simply possible code modifications that might be applicable in a particular point in code. Context actions typically offer a chance to improve the code without dramatically changing it. CA items are displayed with pencil icons.
* _Quick-fixes_ are possible modifications that appear associated with a particular highlighting (i.e., a warning or an error). These typically offer a chance to correct a particular problem in your code. Quick-fix items are displayed with red or yellow light bulb icons (where quick-fixes with red bulbs are meant to fix errors and quick-fixes with yellow bulbs fix warnings, suggestions and hints).

In each case, a context action (CA) or quick-fix (QF) can provide zero or more menu entries for the user to act upon. As of ReSharper 7, these entries can also be hierarchical in nature, i.e. one can have nested/submenu items in addition to top-level ones.

## Bulb Action

Each item provided by a QF or CA is called a _bulb action_. A bulb action is any class that implements the `IBulbAction` interface. This interface has two members:

* `Text` - this property needs to contain the text that is displayed in the menu item.
* `Execute()` - this method is called when the menu item is chosen.

Whether or not to create separate classes for bulb actions (in addition to QF/CA classes) is a design decision. Typically, if your QF or CA only intends to display one item, it may not be necessary.

In addition to ‘plain text’ offered by `IBulbAction`, your bulb action can also provide _rich text_, i.e., text that has some formatting. For example:

![Quick Fix with rich text](rich_text.png)

To support it, your bulb action must also implement the `IBulbItemRichText` interface. This interface has a property called `RichText` that contains a definition of the text shown in the menu, including formatting changes such as bold text or a different text colour. Here’s an example:

```cs
public RichText RichText
{
  get
  {
    var style = new TextStyle(FontStyle.Bold, Color.Red);
    return new RichText("Hello, ").Append("World!", style); 
  }
}
```

Check out the `TextStyle` class for a range of possible options.

## Bulb Menu

A _bulb menu_ is a collection of bulb actions provided by various CAs and QFs, all arranged in a particular list or hierarchy. Bulb menus are created internally by ReSharper, but they should be populated explicitly in the QFs and CAs via the `CreateBulbItems` method.

A bulb menu is represented by the `BulbMenu` class. The contents of this class can be modified in several ways. The simplest way, if you want to add just one item to the root level of the menu is to call a method such as `ArrangeContextAction()` or `ArrangeQuickFix()`. Both of these methods takes one bulb action and puts it at the top level.

A singular call such as `ArrangeContextAction()` has a corresponding method `ArrangeContextActions()` - note the `s` at the end. This basically takes _several_ bulb items and puts them at the top level.

> **Note** the bulb menu doesn’t just support adding ordinary QF and CA items but also other items, but also more complicated items such as refactoring quick-fixes. The fundamental difference between these is the icon that is displayed next to the item.

Now let’s talk about the hierarchical aspects of menus. Essentially, a bulb menu keeps information about its structure in a property called `Groups` which is an enumeration of `BulbGroup` objects. Rather than creating groups, it is recommended that you use the `BulbMenu.GetOrCreateGroup()` method to avoid creating duplicate groups. This method takes an `Anchor` which relates to the location where the group is placed - take a look at the static members of `AnchorsForBulbMenuGroups` for some well-known anchors.

Having acquired or created a group, you can do one of two things:

* Add a single bulb action using the `AddBulbAction()` method. This is precisely what happens behind the scenes in `ArrangeContextAction()` and similar methods.
* Use the `GetOrCreateSubmenu()` method in order to create a submenu for a particular menu.

## Hierarchical Menu (a.k.a. Submenu)

This is where an explanation of submenus is in order. You see, one of the properties of a `BulbGroup` is called `MenuItems` and it contains, you guessed it, a collection of `BulbMenuItem`s. What happens behind the scenes in the `GetOrCreateSubmenu()` method is that a new `BulbMenuItem` is acquired or created and then added to the collection of existing items.

Both the explicit creation of the `BulbMenuItem` and its addition via `GetOrCreateSubmenu()` require you to initialize and pass in a `BulbMenuItemViewDescription` structure. For this structure, you need to define:

* An anchor (as per bulb groups)
* An icon. You can take one of the existing icons in ReSharper or pass `null` to avoid having an icon altogether.
* A rich text definition. Use the `RichText` class or, if you don’t need any formatting, simply cast an ordinary string to `RichText`.


You’ll also note that the `BulbMenuItem` constructor has two additional parameters. The `bulbAction` parameter lets you define a bulb action to execute when this menu item is triggered. The `withSubmenu` parameter defines whether you need to have a submenu for this menu item. If you do, it initializes the `Submenu` property of the `BulbMenuItem` to a new `BulbMenu`.

Brief recap of the way things are structured:

* You have a top-level `BulbMenu` that you can items directly to.
* A `BulbMenu` has one or more `BulbGroup` elements in a `Groups` collection.
* Each group has one or more `BulbMenuItem` elements in a `MenuItems` collection.
* Each `BulbMenuItem` can itself have a `Submenu` property of type `BulbMenu`.

## Context Action

As mentioned previously, a context action is meant to offer an opportunity to change code in a particular context. For example, offering to change a number from hexadecimal to decimal should be a context action - this is a convenience method.

A context action is a class that follows the following rules:

* It implements the `IContextAction` interface
* It is decorated with the `[ContextAction]` attribute
* It has a constructor that takes the CA data provider as a parameter

Let’s start with the CA constructor. As mentioned above, it should have one parameter corresponding to a _data provider_ for the context action. The data provider is how we can get information about where the context action is possibly going to be displayed. In other words, it's the context for the context action. The data provider is a language-specific interface derived from `IContextActionDataProvider`.

Here is an example from a C# context action:

```cs
[ContextAction(Description="Foo",Group="C#",Name="Foo", Priority=1)]
public class OrdinaryContextAction : IContextAction
{
  private ICSharpContextActionDataProvider dataProvider;

  public OrdinaryContextAction(ICSharpContextActionDataProvider dataProvider)
  {
    this.dataProvider = dataProvider;
  }

  // ...
}
```

Now it’s time to implement the interface members. The first, an `IsAvailable()` method is used to check whether this context action is available. If it is, the `CreateBulbItems()` method will be called. If not, it's not called, and no items are added for this action. The `IsAvailable()` method is implemented by looking at the context data provider we were passed in the constructor, and deciding if our action is valid for the current location. There are several properties on the method that are useful, for example, we can get at the current `ITextControl`, `Selection` and `CaretOffset`. More importantly, we can look at the current node in the PSI syntax tree - the `SelectedElement` property gives us the `ITreeNode` of the PSI syntax tree at the current caret offset. We can downcast this to a more specific type, such as `IMethodDeclaration` or `IAssignmentExpression`. We can also get the tree nodes before and after the caret, using the provider's `TokenBeforeCaret` and `TokenAfterCaret`. However, the `GetSelectedElement<T>` method is most useful, because it will walk up the syntax tree looking for a containing node of the correct type. For example, we can use `GetSelectedElement<IMethodDeclaration>(true, true)` to get the containing method declaration, even if the node we're currently on is an expression. We can check to see if the value is null to ensure we're in the right place, and we can walk the contents of the method declaration node to look inside the method body or parameters, etc.

```cs
public bool IsAvailable(IUserDataHolder cache)
{
  return provider.GetSelectedElement<IMethodDeclaration>(true, true) != null;
}
```

The next interface member is the one that populates the bulb menu. The method is defined as

```cs
public void CreateBulbItems(BulbMenu menu)
```

and the idea is that you use the `menu` parameter to define the structure of the menu, as described in the previous section. For instance, the implementation can be as simple as

```cs
public void CreateBulbItems(BulbMenu menu)
{
  menu.ArrangeContextAction(new FooBulbAction());
}
```

if you’ve only got one bulb action that you want to add.

## Quick-Fix

A quick-fix is just like a context action. The only difference is that a quick-fix appears in response to a highlighting. A quick-fix is called this because its purpose is to fix a particular problem, a problem typically found by a daemon and highlighted in code.

The creation of a quick-fix is very similar to the creation of a context action. You need a class that:

* Implements the `IQuickFix` interface
* Is decorated with the `QuickFix` attribute
* Has at least one constructor with one parameter corresponding to a particular highlighting

The constructor parameter that the quick-fix takes must correspond to the highlighting which corresponds to this quick-fix. As far as interface members are concerned, the situation is identical to that of the context action, with the only difference that the `CreateBulbItems()` method takes an extra parameter indicating the `Severity` of the associated highlighting.

Note that `BulbMenu.ArrangeQuickFixes()` requires a set of pairs of bulb actions and severities. Thus, if you keep an internal list of `IBulbAction` elements, you can convert it to a set of pairs using such code as:

```cs
menu.ArrangeQuickFixes(_items.Select(_ => Pair.Of(_, severity)));
```

## Base Classes

In the majority of cases, implementing `IBulbAction`, `IQuickFix` or `IContextAction` is not recommended. Instead, we suggest that you do the following:

* If your QF or CA publishes only a single bulb action, consider inheriting your class from `QuickFixBase` or `ContextActionBase` respectively.
* If you really need to have a separate class for a bulb action, inherit from `BulbActionBase`. This class implements `IBulbAction` and has a lot of useful plumbing necessary for modifying documents.

