# Themed Icons

ReSharper supports theming icons on two axes - application colour, and source code symbol style. In other words, icons can be themed either based on the application colour, or on the style for source code symbols.

The application colour is used to match the theme of the host application, specifically Visual Studio, and is used by icons representing the parts of the application - toolbars, context menus, option pages. The following icon sets are available:

* **Colour** - used when Visual Studio is in the Blue theme.
* **Gray** - for Visual Studio's Light theme.
* **Dark Gray** - for Visual Studio's Dark theme.

ReSharper does not hard code Visual Studio's themes, but analyses the theme colours to detect high and low contrast monochrome themes. If the theme does not appear to be monochrome, ReSharper will use the Colour theme. This helps ReSharper better support custom themes.

The source code symbol style is used by the icons displayed to represent code elements, for example in code completion lists, navigation windows or the file structure window. There are five choices here:

* **Idea** - the icon set used by IntelliJ IDEA.
* **Gray** - matches the icons used by Visual Studio 2012, in the Light theme.
* **Dark Gray** - matches the icons used by Visual Studio 2012, in the Dark theme.
* **Colour** - matches the icons used by Visual Studio 2012, in a colour theme.
* **Legacy Visual Studio** - matches the icons used by Visual Studio 2008.

By default, both of these theme aspects are chosen automatically, meaning the application colour will match the environment, and the source code symbols will match the application colour. However, both can be overridden by the user in the options dialog.

## Identifying the current theme

The current theme is represented by the `IconTheme` class, and can be retrieved using `IThemedIconManager.CurrentIconTheme` (or `ITheming.CurrentIconTheme`, since `ITheming` inherits from `IThemedIconManger`). The `IconTheme` class is a simple value class that maintains an array of `IconThemeAspect` instances, in priority order.

A theme aspect represents the two axes described above - application colour and symbol style. The application colour aspect values are represented by static values on the `GenericIconThemeAspect` class:

```cs
public static class GenericIconThemeAspect
{
  public static readonly IconThemeAspect Color = new IconThemeAspect(...);
  public static readonly IconThemeAspect Gray = new IconThemeAspect(...);
  public static readonly IconThemeAspect DarkGray = new IconThemeAspect(...);
}
```

The `PsiSymbolIconThemeAspect` class holds the values for the source symbol style:

```cs
public static class PsiSymbolIconThemeAspect
{
  public static readonly IconThemeAspect SymbolsIdea = new IconThemeAspect(...);
  public static readonly IconThemeAspect SymbolsVs08 = new IconThemeAspect(...);
  public static readonly IconThemeAspect SymbolsVs11Color = new IconThemeAspect(...);
  public static readonly IconThemeAspect SymbolsVs11Gray = new IconThemeAspect(...);
  public static readonly IconThemeAspect SymbolsVs11GrayDark = new IconThemeAspect(...);
}
```

When inspecting the `IconTheme.Aspects` array, the values should be compared against the values in `GenericIconThemeAspect` and `PsiSymbolIconThemeAspect`.

There may be more than one value for each aspect in the `IconTheme.Aspects` array. These are fallback values, and are there for when a particular aspect value is not recognised or supported. The array is stored in priority order, so each value should be inspected in turn and the first recognised icon should be used.

The symbol style aspects are listed first, so a custom icon set will try to use a specific source symbol style first, then symbol fallbacks, then application colour and colour fallbacks.

## Using the correct icon theme

When retrieving an icon for a given `IconId`, the `IThemedIconManager` will automatically use the current theme, by examining the `IconTheme` and its aspects.

To make sure the icon is updated when the theme changes, either use a "live" image source, or subscribe to the `IThemedIconManager.CurrentIconTheme` observable property, and manually retrieve a new image source from the `IThemedIconManager.Icons` property.

If using a live image source, e.g. `IThemedIconManager.Icons[myIconId].LiveImageSource` or `IThemedIconManager.Icons[myIconId].LiveGdipBitmapScreenDpi`, then the observable property will notify of changes to the image when the theme changes.

Alternatively, use the WPF `ThemedIconViewImage` control or the `ThemedIcon` XAML markup extension to automatically update the image source when the theme changes. See [Consuming Icons](../Icons/ConsumingIcons.md) for more details.

## Creating theme aware icons

When creating theme aware icons, each image should be identified by the theme aspect it applies to. The recognised application colour theme aspect names are:

* **Color**
* **Gray**
* **DarkGray**

And for source code symbols:

* **SymbolsIdea**
* **SymbolsVs11Color**
* **SymbolsVs11Gray**
* **SymbolsVs11DarkGray**
* **SymbolsVs08**

The details of how to specify the aspect name are listed in the [creating compiled icons](../Icons/CreatingCompiledIcons.md) section.
