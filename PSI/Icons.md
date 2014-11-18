# PSI Icons

The PSI needs to be able to display icons to help identify code elements in completion, navigation or find usage result lists. The PSI can display different icons to distinguish between classes, constructors, methods, parameters, etc. It can also show specific icons for CSS class IDs, HTML elements and unit test elements.

While the [Shell provides the `ThemedIconManager` class](../Platform/Shell/Icons.md) in order to load and create icons, it is a low level API. It provides no means for getting the icon for a class, other than explicitly loading `PsiSymbolsThemedIcons.Class.Id`. The PSI provides the `PsiIconManager` to retrieve icons for a specific declared element.

The `PsiIconManager` provides the following methods and properties:

```cs
public abstract class PsiIconManager
{
  public abstract IconId GetDummyImage();
  public abstract IconId GetImage(IDeclaredElement element, PsiLanguageType language, bool drawExtensions);
  public abstract IconId GetImage(DeclaredElementType elementType);
  public abstract IconId GetImage(DeclaredElementType elementType, PsiIconExtension extensions);
  public abstract IconId AttachExtensions(IconId image, PsiIconExtension extensions);
  public abstract IconId KeywordIcon { get; }
  // ...
}
```

The `PsiIconExtension` type is a flags based enum, which provides the modifiers, such as public, private, internal, or abstract, virtual, static, etc. These modifiers are used to create a composite icon, with the base icon representing the element being overlayed with another icon that represents the modifier.

* **`GetDummyImage`** returns a dummy, placeholder image, which is to be used if the icon manager cannot return a more specific image. The pattern for usage is usually:

    ```cs
    var iconId = manager.GetImage(CLRDeclaredElementType.CONSTRUCTOR) ?? manager.GetDummyImage();
    ```

* **`GetImage`** provides several overloads. The first takes an instance of `IDeclaredElement`, and will return an icon specific to that element, for example, given an element for a class declaration, the method will return an `IconId` for the class icon. The `drawExtensions` parameter is true if the element's modifiers (public, private, abstract, etc.) are used to modify the icon.

    The other overloads use an instance of `DeclaredElementType`, which is a metadata class that describes how various declared element types are presented, including presentable name and which icon to use. ReSharper ships with several classes that describe element types, such as `CSharpDeclaredElement`, `CssDeclaredElementType` or `WebConfigDeclaredElementType`. Each class has several static fields that provide instances of the element type, such as `WebConfigDeclaredElementType.WEB_CONFIG_TAG_NAME`.

* **`AttachExtensions`** will add the expected modifier overlays to the passed icon, marking it as private, protected, abstract, static, etc.

* **`KeywordIcon`** simply returns the `IconId` of the icon representing keywords.

Typically, the icon manager will return an icon when `GetImage` is called. If it returns `null`, use the image returned by `GetDummyImage` instead.

## Extending the icon manager

By default, the icon manager will try to return a specific icon for a given declared element. If it can't find one, it returns the icon from the element's `DeclaredElementType`. In order to find a specific icon, it asks an extensible collection of icon providers, which all implement the `IDeclaredElementIconProvider` interface:

```cs
public interface IDeclaredElementIconProvider
{
  IconId GetImageId(IDeclaredElement declaredElement, PsiLanguageType languageType, out bool canApplyExtensions);
}
```

ReSharper ships with a number of language specific providers, such as the C# provider that checks to see if the declared element is a C# extension method, indexer property or finalizer (all of which would otherwise be classed as a simple method), and returns back a specific icon in each case. If the element isn't one of these, it returns null. It also returns `true` in the `canApplyExtensions` parameter to indicate that this element can have modifiers, based on `PsiIconExtension`.

Furthermore, ReSharper ships with a colour icon provider, which recognises type members that are colour properties, and returns a `ColorIconId` to match. This provider returns `false` in `canApplyExtensions`. ReSharper also ships a unit test icon provider, which checks to see if the declared element is part of a unit test, and returns an appropriate unit test icon.

The collection can be extended by creating a class that implements `IDeclaredElementIconProvider` and is decorated with the `[DeclaredElementIconProvider]` attribute. This scopes the class to PSI-shared component lifetime (that is, the same lifetime as a `ShellComponent`). Should the class need to have a narrower lifetime, such as `[SolutionComponent]`, it is possible to call `PsiIconManager.AddExtension` in the provider's constructor, passing in it's own `Lifetime` for cleanup. The unit test icon provider does this, as it depends on components that have solution lifetime scope.

The `IDeclaredElementIconExtensionProvider` can be implemented to extend the process for deciding what modifiers should be applied to the icon (private, public, etc.). The class should be decorated with the `[DeclaredElementIconExtensionProvider]` attribute, or can also be registered by calling `PsiIconManager.AddExtension` if the component has different lifetime requirements. This provider is currently implemented for CLR and C++ elements.

## Loading icons manually

It is not recommended to load icons directly, but they are available in the [`ThemedIconManager`](../Platform/Shell/Icons/ConsumingIcons.md). Simply use the icon IDs defined in the `PsiSymbolsThemedIcons` class, and use [`CompositeIconId.Compose`](../Platform/Shell/Icons/IconTypes.md#composite-icons) method to apply modifiers. There are language specific icons in other classes, such `PsiCssThemedIcons` or `PsiBuildScriptsThemedIcons`.

The `PsiIconManager` provides a more flexible and extensible way of getting the correct icon.
