[//]: # (title: Options Pages)

 >  This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.
 >
 {type="warning"}

An options page allows a plugin developer to add controls to the ReSharper Options dialog. This is typically used to let the user specify various plugin settings. The plugin writer can add an unlimited amount of option pages to the dialog, and the dialogs can be nested in any of the options groups.

Let us now discuss the way in which option pages are defined.

## Making an options page

Making an options page is surprisingly easy. You begin by defining a class which will house the options page. This class should be made to implement `IOptionsPage`, and should be decorated with the `OptionsPage` attribute.

The `OptionsPage` attribute requires the plugin author to provide the following parameters:

* The page ID. This ID can be specified as a constant field inside the class. The page ID is a string which uniquely identifies this particular options page.
* The name of the page. This is the text that will appear in the left-hand tree on the Options page as well as in the title and navigation elements.
* The image. This refers to the glyph that appears next to the item in the tree and is specified as a type (e.g., `typeof(OptionsPageThemedIcons.SamplePage)`). See [Icons](Shell_Icons.md) for more information.

In addition, you may specify the following optional parameters:

* `ParentId` lets you define the ID of the section or element which serves as this page's parent. If you want the parent to be one of the known ReSharper items, look inside the `JetBrains.UI.Options.OptionsPages` namespace for the corresponding pages, and then use their `Pid` as this parameter. For example, for an options page to appear in the Environment section, you specify the `ParentId` of `EnvironmentPage.Pid`.
* `Sequence` lets you define the location of the item you are inserting in relation to the other items. Items are placed in order, so the higher this value, the further this page will be in the list of items. Of course, to accurately position the item, you need to know the `Sequence` value of its siblings. Luckily, this information is available in the metadata.

Having specified the attributes, your class will appear as follows:

```csharp
[OptionsPage(PID, "Sample Page", typeof(OptionsPageThemedIcons.SamplePage), ParentId = ToolsPage.PID)]
public partial class SampleOptionPage : IOptionsPage
{
  private const string PID = "SamplePageId";
}
```

## Injecting dependencies

Options pages are created by the Component Model, which means you can inject dependencies via your constructor parameters. Your constructor should take at least the following two parameters:

* `Lifetime`, which controls the lifetime of this page.
* `OptionsSettingsSmartContext`, the settings context that you can use to bind UI elements.

Both of these values need to be injected because they are required for binding particular settings to UI elements. If you are inheriting from `AOptionsPage`, you will also need to inject `IUIApplication` to pass into the base class constructor.

In addition to these values, you may inject any other available component into the service. Note that if you implement `IOptionsPage` on a user control, you should ensure that the generated default constructor is replaced with the constructor you wish the component model to inject dependencies for.

## Defining the UI

You can define the UI for your options page using either Windows Forms or WPF. Whichever option you choose, all you have to do to actually present the UI is to initialize it and assign it to your option page's `Control` variable.

 >  whichever UI framework you choose, your application _must_ reference the WPF assemblies. The compiler will warn you about this if you start using the `EitherControl` type without adding appropriate references.
 >
 {type="note"}

To create an options page using Windows Forms, simply create a `UserControl` and assign it to the `Control` property. Please note that since multiple inheritance is impossible, the only way to keep the options page class and the `UserControl` class one and the same is as follows:

* Inherit from `UserControl` and implement the `IOptionsPage` interface.
* Decorate the control with the `OptionsPage` attribute as described above.
* Implement the read-only property `Control`, returning the value of `this`.

To create an options page using WPF, simply define your UI in terms of WPF elements and then assign the `Control` property accordingly. You can specify any WPF control, e.g., `Grid`, as the page control.

Needless to say, it is entirely possible to use the `WindowsFormsHost` class to host Windows Forms controls on a WPF options page. The mechanism which binds the controls to the settings works for both WPF and Windows Forms. (Of course, if you implement `IOptionsPage` manually, you can simply assign properties manually without using bindings at all.)

## Working with Settings

The `OptionsSettingsSmartContext` class that we inject has several `SetBinding()` methods that let us tie together settings and controls. These bind methods have two generic arguments - the name of the settings class, and the type of property that is being saved. In the case of WPF, you would specify:

* The property that is being assigned on exit. Defined as a lambda expression (e.g., `x => x.Name`).
* The name of the control that the property is being read from.
* The dependency property that is being read.

For example, here is how one would bind a WPF text box for a username to a corresponding setting:

```csharp
settings.SetBinding(lifetime, (GitHubSettings s) => s.Username, usernameBox, TextBox.TextProperty);
```

The situation with WinForms is a bit more trickly - there are no dependency properties to be used, so we use the `WinFormsProperty` helper class. This helper class has a single method, `Create()`, that creates an object of type `IProperty<T>` (where `T` is the property type). To create the property, it requires the following parameters:

* The `Lifetime` of the calling component. This should be obvious, since the 'proxy property' should only live as long as it is needed. This does, of course, imply that you _must_ inject the `Lifetime` into the constructor.
* The class to take data from. In actual fact, though in the case of WinForms you'll probably provide the corresponding control, this doesn't have to be a control _per se_ - it can be practically any object. After all, the `WinFormsProperty` class does not use any WinForms-specific code.
* A lambda expression indicating which property of the aforementioned class that is to be used.

Thus, the call to bind a WinForms-based password box to a setting becomes as follows:

```csharp
var property = WinFormsProperty.Create(lifetime, passwordBox, box => box.Text, true);
settings.SetBinding(lifetime, (GitHubSettings s) => s.Password, property);
```
