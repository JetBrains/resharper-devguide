[//]: # (title: Consuming Icons)

ReSharper provides several ways to consume icons), from WPF controls to display the icon, to APIs to get a Windows Forms compatible bitmap or WPF `ImageSource`.

The main entry point to the API is the `IThemedIconManager` interface, which can be imported as a constructor parameter on a `[ShellComponent]` or `[SolutionComponent]`. It provides the following members:

```csharp
public interface IThemedIconManager
{
  IProperty<IconTheme> CurrentIconTheme { get; }
  ThemedIconManagerIcons Icons { get; }
  IThemedIconManagerRawApi GetRawApi();
}
```

The `CurrentIconTheme` property will return an observable property that contains the current icon theme, which can be retrieved for inspection, or subscribed to, in order to know when a theme has changed. See the [icon theming section](Theming_Icons.md) for more details on `IconTheme`.

The `GetRawApi` method is intended for internal use.

The Icons property returns an instance of `ThemedIconManagerIcons`, which can be used to get `ThemedIconLoader`, which in turn is responsible for loading and caching images.

```csharp
var loader = manager.Icons[myIconId];
```

Once you have a `ThemedIconLoader`, the following properties will retrieve a bitmap and `ImageSource`, either based on the current theme, or as a live, observable property that updates when the theme changes.

* **`CurrentGdipBitmapScreenDpi`** returns a `System.Drawing.Bitmap` of the icon for the current theme, for use in Windows Forms. The icon is rendered at the current screen DPI, so will work correctly in high DPI scenarios. It is recommended over the `CurrentGdipBitmap96` property, which renders the icon at 96 DPI, which will not look good in high DPI.
* **`LiveGdiBitmapScreenDpi`** returns an observable `IProperty<Bitmap>` which will notify of new values when the theme changes. The bitmap is rendered at the screen DPI and should be favoured over `LiveGdipBitmap96` which is only rendered at 96 DPI, and will not look good in high DPI.
* **`CurrentImageSource`** returns a WPF `ImageSource` of the icon for the current theme. This can be a vector image, or a bitmap that will be correctly rendered when used in a WPF window.
* **`LiveImageSource`** returns an observable `IProperty<ImageSource>` which will update whenever the image changes, due to theme changes.

## WPF controls

The `ThemedIconViewImage` class is a WPF element that derives from `Image`, and can be used as a drop in replacement. It will set up a live binding from an icon ID to the `Source` property. If the icon ID changes, or if the icon represented by the ID changes (due to a theme change), the binding automatically updates the `Source` property and the new image is displayed.

The icon ID can be specified either in the constructor, or by setting the `IconId` property. Alternatively, the icon ID can be bound against the `DataContext` property, allowing for automatic updates:

```csharp
public UIElement GetThemedIconViewImage(Lifetime lifetime, IProperty<IconId> liveIconId)
{
  var image = new ThemedIconViewImage();
  // Wire up the live icon ID property to push the value into the DataContext,
  // which in turn causes the correctly themed icon to be displayed. The
  // property binding is automatically cleared when the Lifetime terminates
  liveIconId.FlowInto(lifetime, image, DataContextProperty);
  return image;
}
```

If using compiled icons, ReSharper provides the `ThemedIcon` markup extension to provide live binding to an `Image` element's `Source` property. See below for details.

## Using compiled icons

The most common icon type is the compiled icon, which is compiled as a resource and embedded in an assembly. Compiled icons are identified with an instance of the `CompiledIconId` type. Typically, these icon ID instances are automatically generated, and can be found as the `Id` static property of a class named after the icon, itself implemented as a nested class named after the icon group.

For example, you can get the icon ID for the To Do Explorer option page using:

```csharp
var iconId = TodoItemsThemedIcons.TodoOptions.Id;
```

It is also possible to bind a compiled icon to an image source in WPF, using the `ThemedIcon` markup extension. The extension will create a live binding, which automatically updates the image when the icon represented by the compiled icon ID changes, usually in response to a theme change.

```xml
<Image Source="{icons:ThemedIcon myres:TodoItemsThemedIcons+TodoOptions}" />
```

Note that the parameter to the extension is the class that holds the `CompiledIconId` instance, and not the `Id` property. This is because the XAML requires a compile time value, rather than a runtime instance.

A similar requirement is necessary when referencing an icon in an attribute, for example with `OptionsPageAttribute`:

```csharp
[OptionsPage(PageId, "To-do items", typeof(TodoItemsThemedIcons.ToDoItemsPage))]
public class TodoItemsPage : IOptionsPage
{
  // ...
}
```

The `OptionsPageAttribute` class converts the `Type` to an `IconId` using the `CompiledIconClassAttribute.TryGetCompiledIconClassId` static method:

```csharp
var iconId = CompiledIconClassAttribute.TryGetCompiledIconClassId(typeOfIcon, OnError.LogException);
```

Once the `IconId` is known, the standard icon loader API discussed above can be used to get a bitmap or `ImageSource`. A shortcut is to use the `GetIcon` extension method for `IThemedIconManager` to return a `ThemedIconLoader` which can be used to get the icon image.

```csharp
IProperty<ImageSource> liveImage = themedIconManager.GetIcon<TodoItemsThemedIcons.ToDoItemsPage>().LiveImageSource;
```

## Discovering compiled icons

ReSharper ships with over 1000 compiled icons, so finding an icon to use can be tricky. A significant number of the icons are not intended to be used by extensions - they are icons for tool windows, actions or other UI elements that are not relevant to extensions. It is recommended to create your own icons where possible, but if you wish to reuse existing icons, make sure they are intended for the same purpose (for example, do not use the "unit test pass" icon to represent success in an extension).

Should you wish to reuse existing icons, you can make use of an Internal feature that allows you to browse all of the icons registered with the `ThemedIconManager`. Start ReSharper in Internal mode, and use the ReSharper → Internal → Windows → Show Themed Icons menu option. The tool window that opens will show all available icons, either in a tile view, flat list, or grouped by owner. This grouped option is very useful, as it will group the icons by their owner - in the case of compiled icons, this will give an indication of the name of the compiled icon ID class (the name is actually the name of the "icon pack", or the XAML file that contains the definition of the icon, but that name is usually the same as, or very similar to the icon ID class name).

When displaying as a flat or grouped list, the icons are searchable, and typing a value will filter out any icons that don't match the search text.

## PSI symbols

While it is possible to use the icon manager to get icons for PSI elements such as class, method or parameter, it is highly recommended to use the `PsiIconManager` class instead. See the section on [icons in the PSI](PSI_Icons.md) for more details.