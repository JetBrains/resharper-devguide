# Custom Icon Types

It is possible to create a custom icon type, if the existing item types do not meet requirements. For example, the image could be retrieved from the filesystem, rather than compiled into an assembly.

To create a custom file type, you need to create a class that derives from `IconId`. This class identifies your image, and should provide enough information to be able to create an image. It should be a value type, and must override the various `Equals`, `CompareTo` and `GetHashCode` methods to be able to properly compare two instances as the same.

Each `IconId` is associated with a class that implements `IIconIdOwner`, and is marked with the `[ShellComponent]` attribute. This class is responsible for taking the information in the `IconId` derived class and retrieving the icon.

```cs
public interface IIconIdOwner
{
    Type IconIdType { get; }
    ImageSource TryGetImage(IconId iconid, IconTheme theme, IThemedIconManagerPerThemeCache themedIconManagerPerThemeCache, OnError onerror);
}
```

The `IconIdType` member should return the `Type` of the derived `IconId` that the owner handles. When trying to get an image from an `IconId`, this type is used to lookup which class can handle the icon.

The `TryGetImage` method is called to retrieve the image. The owner class should downcast the passed icon ID to the type of `IconId` it supports, extract the information and use that to retrieve the image. It should always return something. If it can't find an image, or if an error happens, it should return `IconPlaceholder.IconPlaceholderAvalon`, which is a WPF `ImageSource` that represents a broken icon (an icon with a cross).

In the case of an error, the class should pass any exception to the `Handle` method on the `onerror` parameter, and return the placeholder image.

## Using existing icons

If the icon owner needs to use other icons, perhaps to apply an overlay (although the `CompositeIconId` should handle this), it can use the `IThemedIconManagerPerThemeCache` instance that gets passed in. This interface is a cache of existing icons for the current theme, and can be used to get an icon image, as a WPF `ImageSource` or a `System.Drawing.Bitmap` for a particular DPI.

```cs
public interface IThemedIconManagerPerThemeCache
{
    ImageSource GetIconImageSource([NotNull] IconId iconid);
    Bitmap GetIconGdipBitmap([NotNull] IconId id, RasterizationResolution resolution);
    ImageSource TryGetIconImageSource([NotNull] IconId id);
}
```

The images are created on demand, or pulled from the in-memory cache. The `GetIconImageSource` will return the appropriate image, or will return the placeholder. The `TryGetIconImageSource` method will return `null` if it can't find the requested icon.

## Theming

It is up to the icon owner to handle theming. Some icons don't partake in theming, for example, `ColorIconId` returns a solid block of colour, which is not appropriate for theming.

When asked for an icon, one of the items to be passed in is an instance of `IconTheme`. An icon theme is made up of a number of aspects, represented by an array of `IconThemeAspect` instances. There are currently two types of aspect that are recognised - "Generic" and "PsiSymbol".

The generic aspect represents the colour scheme used by the current theme - colour, gray or dark gray. This can be set explicitly by the user, or set to "automatic", where the aspect is chosen based on the Visual Studio theme. ReSharper analyses the theme to see if it is monochrome, and to see if it is light or dark. For the Blue theme, this gives `GenericThemeAspect.Color`. For the Light theme, `GenericThemeAspect.Gray`, and the Dark theme gives `GenericThemeAspect.DarkGray`.

The "PsiSymbol" aspect is used to choose which icon set to use for PSI elements such as methods, properties, etc. in code completion lists and file structure windows and so on. Again, this value can be set by the user, but by default is set to automatic. ReSharper will look at the current Visual Studio theme and version, and choose the appropriate symbol set:

* `PsiSymbolIconThemeAspect.SymbolsIdea` provides symbols based on IntelliJ IDEA
* `PsiSymbolIconThemeAspect.SymbolsVs08` provides symbols used in Visual Studio 2008
* `PsiSymbolIconThemeAspect.SymbolsVsColor` provides colour symbols as used in Visual Studio 2012 and later
* `PsiSymbolIconThemeAspect.SymbolsVs11Gray` provides monochrome VS2012 symbols for use on a light background
* `PsiSymbolIconThemeAspect.SymbolsVs11GrayDark` provides monochrome VS2012 symbols for use on a dark background

When the `IIconIdOwner` class needs to return a themed icon, it should look at `IconTheme.Aspects` for aspects it recognises. The array is in priority order, and there may be more than one entry for each aspect kind, to allow for fallbacks. The icon owner should compare the aspect against the known static aspects listed above. When it recognises that aspect, it should provide the appropriate icon.

How the icon owner chooses to get a theme aspect specific icon is up to the implementation of the icon owner. `CompiledIconId` icons embed several different versions of the icons into an assembly's resources, and the owner simply retrieves the version of the icon that matches the aspect. The `ShellFileIconIdOwner` on the other hand, retrieves a file type icon from the operating system, looks for the `GenericIconThemeAspect.Gray` theme aspect and makes the icon monochrome using `ColorManagement.MakeMonochrome`.

```cs
BitmapSource bmp = GetFileIcon(iconid.ExtensionWithDot);
if (theme.Aspects.Contains(GenericIconThemeAspect.Gray))
  return ColorManagement.MakeMonochrome(bmp);
```

Both the Light and Dark themes will contain the `Gray` aspect, but the priority ordering is different. This allows for fallback icons.

> **NOTE** the `IIconIdOwner` class does not need to perform caching. ReSharper will only call the class once per icon per theme. However, the caches are cleared each time the theme changes, so the owner will be called again.


