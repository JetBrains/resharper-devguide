---
---

# Icon Types

The most common kind of `IconId` is `CompiledIconId`. This represents icons that have been compiled into an assembly's resources. ReSharper also supports several other types of icons:

* **`CompiledIconId`** represents icons that have been compiled into an assembly's resources.
* **`ColorIconId`** represents an icon that is a solid colour.
* **`CompositeIconId`** is an icon that is composed of other `IconId` icons.
* **`EffectOnIconId`** creates an icon based on another `IconId`, applying alpha, made monochrome based on a tint colour, and/or with a blur radius.
* **`ShellFileIconId`** is a file type icon for a given extension.
* **`AnimatedIconId`** and **`RotatedIconId`** represent icons that are animated.

The icon types don't apply theming, and when creating a new icon based on an existing icon, will create a WPF `ImageSource` that uses the existing icon, meaning WPF will be responsible for scaling the base icon.

## Color icons

An icon representing a solid colour can be defined by creating a new instance of `ColorIconId` and passing in a `Color`. The resulting icon will be a solid square of that colour.

```csharp
var iconId = new ColorIconId(Color.Red);
```

## Composite icons

The `CompositeIconId` will combine other icons into a single image, useful for applying overlays. To create a composite icon, call one of the static `Compose` methods, passing in the `IconId` of the icons to combine. The images are composed in left to right order. That is, the `Compose` method's leftmost argument being applied first, and subsequent arguments are applied on top.

```csharp
var iconId1 = new ColorIconId(Color.Red);
var iconId2 = ProjectModelThemedIcons.LayerSolutionShared.Id;
var compositeIconId = CompositeIconId.Compose(iconId1, iconId2);
```

A more realistic example shows how to create an icon that represents a private method (although this is unlikely to be required in user code, as the PSI can provide this icon automatically for a given declared element):

```csharp
var iconId1 = PsiSymbolsThemedIcons.Method.Id;
var iconId2 = PsiSymbolsThemedIcons.ModifiersPrivate.Id;
var compositeIconId = CompositeIconId.Compose(iconId1, iconId2);
```

## Applying effects to icons

The `EffectOnIconId` creates a new `IconId` based on an existing icon, but with various effects applied. The new icon can have an alpha opacity value applied, turn the icon monochrome, apply a blur radius, or any combination of the three. Alpha transparency is applied as a double value from 0 to 1, where 1 is no transparency and 0 is fully transparent. Applying the blur radius requires passing in a double value that represents the radius in WPF's Device Independent Units (1/96 of an inch).

When making the icon monochrome, if the base icon is a WPF vector image, the brushes associated with the vector geometries are replaced with the given colour. When the base icon is a bitmap, the bitmap is rendered and the monochrome colour is applied.

```csharp
var baseIconId = ProjectModelThemedIcons.LayedSolutionShared.Id;
// Apply a 0.7 opacity, convert to red and apply a radius blur with a radius of 5
var iconId = new EffectOnIconId(baseIconId, 0.7, Color.Red, 5);
```

> **NOTE** `EffectOnIconId` is not intended to be used for theming. Icon types are responsible for their own theming, and should provide icons that are already theme aware. This icon type manipulates an existing icon, which will already be themed if it is appropriate to do so (e.g. `ColorIconId` icons won't be themed, and it's not appropriate for them to be themed).

## File type icons

Icons for file types can be retrieved using the `ShellFileIconId` class. These icon IDs ask the operating system for the icon to represent a given file extension (the dot should be included), at a given file size. The returned icon is also converted to monochrome, if the current theme is a monochrome theme (such as Visual Studio's Light or Dark theme. The Blue theme is not, and the icon is not converted if this is the current theme).

```csharp
var smallIconId = new ShellFileIconId(".cs", ShellFileIconId.IconSize.SmallIcon);
var largeIconId = new ShellFileIconId(".cs", ShellFileIconId.IconSize.LargeIcon);
```

Small icons are 16×16 and large icons are 32×32. Since the icons are raster images, they will not scale well, so try to use the appropriate size for the use case. Note that the resulting image does get created for the current DPI, so it will still display correctly in high DPI scenarios.

> **NOTE** This icon type will convert the icon to monochrome, depending on the current theme. It is up to the icon type to apply appropriate theming. Most other icon types do not need to do this, as they are either manipulating existing icons that are already themed, or because the icon they produce should be displayed as-is. The `CompiledIconId` class will produce theme aware icons, by choosing between multiple sets of icons.

## Animated icons

The `AnimatedIconId` and `RotatedIconId` classes are used to get a static image for scenarios where animation is not supported. The `AnimatedIconId` is simply a base class, and `RotatedIconId` is used to identify and provide another `IconId` as a static icon.

If a `RotatedIconId` is added to a WPF UI using the `ThemedIconViewImage` control (which can be used instead of the WPF `Image` element), then the static image is continuously rotated by a WPF animation, specified in `RotatedIconId.AnimatedRenderTransform`. This is how the animated icons in the unit test runner are rendered.

For Windows Forms, the individual frames of a rotated animation can be retrieved using `RotatedIconId.GdipBitmapFrames.GetAnimationFrameGdipBitmap`. The `PresentableItemRenderer` class makes use of this to render animation frames for `IPresentableItem` objects that include frames.

Alternatively, animation can be achieved by creating icons for individual frames and updating an `IProperty<IconId>` with each frame. Again, this can be used with `ThemedIconViewImage`, which will display each frame as a static image, and update when the icon changes for each frame. This is how the solution wide analysis round icon in the status bar is handled.

```csharp
public UIElement GetIconView(Lifetime animationLifetime, IProperty<IconId> animatedIcon)
{
  var image = new ThemedIconViewImage();
  // Bind the property's value to the ThemedIconViewImage's DataContextProperty
  // When the icon changes, the DataContextProperty is updated, and the image is redrawn
  // The Lifetime dictates when the property binding should be disposed
  animatedIcon.FlowInto(animationLifetime, image, DataContextProperty);
}
```
