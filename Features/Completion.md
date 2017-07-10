---
title: Code Completion
---

Just like Visual Studio, ReSharper implements its own form of _code completion_ (which Visual Studio calls IntelliSense) that is used to provide various helpers when typing code. Unlike Visual Studio, however, ReSharper implements three different varieties of code completion that plugin developers need to be aware of. These are:

* **Symbol completion**, which is the closest analogy to IntelliSense in terms of operation. This is essentially the mechanism that completes the names of symbol identifiers as they are being typed into the editor. Symbol completion is typically invoked with the `Ctrl+Space` shortcut.
* **Smart completion** is a code completion mechanism which attempts to narrow down the list of suggested items given the particular context. For example, when calling a method that takes a `string`, pressing `Ctrl+Alt+Space` while entering the parameter to the method will present the list of all `string` typed identifiers available at the current scope.
* **Import symbol completion** is a means of completion that is used to complete identifier names even if such identifiers have not been imported with a `using` statement. Import symbol completion is invoked with `Shift+Alt+Space` and lets the user quickly add both the identifier name and a `using` statement.

Let’s begin our overview of these mechanisms by looking at the root interface of code completion, `ICodeCompletionItemsProvider`.

## ICodeCompletionItemsProvider

The `ICodeCompletionItemsProvider` is an interface for any type that wants to provide code completion information to ReSharper. We won’t discuss its methods, since you’ll most likely be inheriting from a derived abstract class, but what’s important to note here is that all of its methods take as a parameter an `ISpecificCodeCompletionContext` that is used to make decisions on what completion items are available.

## ItemsProviderOfSpecificContext

Rather than implementing `ICodeCompletionItemsProvide` directly, it makes more sense to inherit from the `ItemsProviderOfSpecificContext<TContext>` class. This class is the one that gets inherited by various ReSharper mechanisms. The two most common interface methods that providers override are:

* `IsAvailable()` - this method determines whether the lookup items (i.e., items inserted into the completion list) are in fact added.
* `AddLookupItems()` is the method that actually adds the items.

Let’s look at these in more detail. First, it’s important to note that the `TContext` generic parameter that you specify will be the first parameter in both of the above methods. This generic parameter typically relates to the language you’re supporting, so that for example, if you want to support code completion in C#, you would inherit from `ItemsProviderOfSpecificContext<CSharpCodeCompletionContext>`.

Once you know what context you have, you can begin to use it in the aforementioned methods. For example, when checking if the completion items are available, you can perform a check similar to the following:

```csharp
protected override bool IsAvailable(CSharpCodeCompletionContext context)
{
  if (context.BasicContext.CodeCompletionType != CodeCompletionType.SmartCompletion)
    return false;

  return true;
}
```

The above check ensures that the items are only available if the type of completion is Smart.

The second method, `AddLookupItems()`, takes two parameters: the smart completion context and a `GroupedItemsCollector`. The second parameter is of particular importance since it is through this parameter that completion items are provided.

To allow adding of items, `GroupedItemsCollector` provides a set of methods such as `AddToTop()` and `AddAtDefaultPlace()`. All of these methods take an `ILookupItem` as a parameter. This interface is by itself fairly complicated, but luckily a number of lookup item factories are available in order to facilitate the process in common scenarios. For example, the `CSharpCodeCompletionContext` class has  a property called `LookupItemsFactory` which yields an instance of a `CSharpLookupItemFactory`. You can use this factory to create different lookup items. Putting it all together, here is an example of how you can add a lookup item to a collector:

```csharp
protected override bool AddLookupItems(CSharpCodeCompletionContext context, GroupedItemsCollector collector)
{
  collector.AddAtDefaultPlace(context.LookupItemsFactory.CreateTextLookupItem("true", "bool", true));
}
```

The `CreateTextLookupItem()` method used above simply creates and initializes a `TextLookupItem` containing specific text.

## CSharpItemsProviderBase

If we go off looking down the hierarchy of the `ItemsProviderOfSpecificContext`, we’ll eventually find language-specific classes such as `CSharpItemsProviderBase`. This class, which also takes a context generic parameter, is the kind of class that one would actually use to build extensions to code completion.

Here’s a very simple example: let’s suppose that we want to add the string _hello_ as a code completion item to any C# file regardless of the position of the caret. In other words, the option _hello_ will be available practically everywhere. To implement this, we first make a class that inherits from `CSharpItemsProviderBase`, i.e.,

```csharp
[Language(typeof(CSharpLanguage))]
public class MyCodeCompletion : CSharpItemsProviderBase<CSharpCodeCompletionContext>
{
}
```

Then, we override the `IsAvailable()` method to only allow our code completion item to appear in automatic completion (and not in smart or type symbol completion):

```csharp
protected override bool IsAvailable(CSharpCodeCompletionContext context)
{
  return context.BasicContext.CodeCompletionType == CodeCompletionType.AutomaticCompletion;
}
```

Finally, we override the `AddLookupItems()` method and add the simplest possible statement to include the string "hello" in the code completion popup list:

```csharp
protected override bool AddLookupItems(CSharpCodeCompletionContext context, GroupedItemsCollector collector)
{
  collector.AddAtDefaultPlace(context.LookupItemsFactory.CreateTextLookupItem("hello"));
  return base.AddLookupItems(context, collector);
}
```

And that’s it - you have a code completion provider that adds an extra element when working in C# context.

## Generative Completion

One form of code completion that made its appearance in ReSharper 8 is called _Generative Completion_. The idea of generative copmletion is to use code completion for code generation as a quicker, more direct alternative to, day, the _Generate_ menu.

One very visible example of generative completion is the `ctorp`, `ctorf` and `ctorfp` templates available in C# classes. This completion mechanism is capable of inserting constructors that initialize either all fields of a class, all properties, or both.

The class responsible for providing these items to the code completion mechanisms is called `ConstructorRule`. Let’s take a look at how it works. This class has two features:

* It is decorated with `[Language(typeof(CSharpLanguage))]`, indicating that it’s applicable to C# code.
* It inherits from `ItemsProviderOfSpecificContext<CSharpCodeCompletionContext>`.

## Pre-requisites

After inheriting from the aforementioned provider, we can override members to customize what, if anything, gets added to the list of completion items. Before we do that, we can also customize other aspects. For example, we can select which types of completion that we support. For example, the check on the `ctor` items is similar to the following:

```csharp
protected override bool IsAvailable(CSharpCodeCompletionContext context)
{
  var type = context.BasicContext.CodeCompletionType;
  if (type == CodeCompletionType.AutomaticCompletion ||
      type == CodeCompletionType.BasicCompletion) return !context.IsQualified;
  else return false;
}
```

The above means that that completion will only work in automatic or basic completion, and that it won’t work if the context is qualified (i.e., `foo.ctorp` won’t complete).

## Checking the location

Before adding a completion item, you want to check that you’re in the right location in code. This is likely done in the overridden `AddLookupItems` method, with the only difference that, unlike in e.g., context actions, you don’t get a context that lets you directly figure out the code element you’re in. Instead, you have to do something like this:

```csharp
protected override bool AddLookupItems(CSharpCodeCompletionContext context, GroupedItemsCollector collector)
{
  ITreeNode node = TextControlToPsi.GetElement<ITreeNode>(context.BasicContext.Solution, context.BasicContext.TextControl);
  if (node == null)
    return false;
}
```

From an `ITreeNode` you can move up the ranks until something like an `IClassBody`. Sooner or later, you’ll get to the point where, given applicability, you want to add your completion items. In the case of `ctor` code completion items, the code looks something like this:

```csharp
var item = new GenerateConstructorLookupItem("ctorf", fields.OfType<IXmlDocIdOwner>(), psiIconManager);
item.InitializeRanges(context.CompletionRanges, context.BasicContext);
collector.AddAtDefaultPlace(item);
```

In the above, `psiIconManager` is a solution component that can be acquired from the solution. At any rate, all that happens is you add the items you want to appear to the `collector`, so that they are subsequently displayed.

## Lookup Item

The lookup item for generative completion is quite simply a class that inherits `TextLookupItem`, and there is nothing special apart from it aside from the fact that it generates a rather large amount of text. One thing to point out is that generative completion items are typically shown with emphasis:

```csharp
protected override RichText GetDisplayName()
{
  RichText displayName = new RichText(myName);
  LookupUtil.AddEmphasize(displayName, new TextRange(0, displayName.Length));
  return displayName;
}
```

All the interesting things happen inside the `Accept()` method override. First of all, the identifier under the caret gets wiped out:

```csharp
IIdentifier identifierNode = TextControlToPsi.GetElement<IIdentifier>(solution, textControl);
IPsiServices psiServices = solution.GetPsiServices();
if (identifierNode != null)
{
  using (var cookie = new PsiTransactionCookie(psiServices, DefaultAction.Rollback, "RemoveIdentifier"))
  using (new DisableCodeFormatter())
  {
    using (WriteLockCookie.Create())
      ModificationUtil.DeleteChild(identifierNode);
    cookie.Commit();
  }
}
psiServices.Files.CommitAllDocuments();
```

Once this is done, you have a variety of choices. If you’re replicating a _Generate_ item, simply initialize an `IGeneratorWorkflow` (remember it’s `IDisposable`), provide the input elements and call `GenerateAndFinish()` to inject the code.

If, on the other hand, you just want to inject arbitrary code at a particular location, that’s not a problem either. For example, instead of doing `ModificationUtil.DeleteChild()`, you can create an entirely new construct that you want to inject (using, e.g., `CSharpElementFactory` and the like) and then simply call `ModificationUtil.ReplaceChild()` instead.

