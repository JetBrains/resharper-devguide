[//]: # (title: Creating Compiled Icons)

The most common type of icon is the compiled icon), represented by `CompiledIconId`. These icons are embedded into assembly references at compile type, as XAML resources.

The ReSharper SDK includes MSBuild build actions and a custom task that will generate icons from either XAML or PNG files. The custom task generates two files - a new XAML file containing a `DrawingBrush` resource for each image, and a C# file providing the `CompiledIconId` instances, and pointers to the new XAML resource dictionary. Once the files have been generated, they should both be added to the Visual Studio project.

## Themed icons

Themed icons are supported by including multiple images in the generated XAML file, one version for each theme aspect supported. Each image is annotated with its own name and the theme aspect that it is intended for, so themed icons will have multiple images with the same name, but different theme aspect names.

When retrieving an icon, the aspects from the current theme are enumerated in priority order, and the first matching image for the icon is used. If there are no images that match any of the theme aspects, the fallback image is used, if available. A fallback image is an image that is created without a theme aspect. If there are no fallback images, an arbitrary themed image for the icon is used.

See the [theming section](Theming_Icons.md) for more details about theming and theme aspects.

## XAML icons

XAML images are the preferred format for icons, as they scale better for larger sizes and higher screen DPI resolutions.

The input XAML file should be a `ResourceDictionary` containing a number of `DrawingBrush` resources, all marked with an `x:Key` attribute that uniquely identifies the icon by name. The file should be added to the Visual Studio project and have its build action set to `ThemedIconsXamlV3`.

Each brush can contain multiple individual images, one for each supported theme aspect. The images are placed horizontally, and, rather than conforming to a strict grid, the custom task will automatically discover distinct, non-overlapping images. These images are saved to the new, generated XAML file, again as a `ResourceDictionary`, with each theme image being saved as its own individual `DrawingBrush`.

The new XAML images need to be annotated, with both the name of the icon and the name of the theme aspect it is intended for. The theme aspect is retrieved from MSBuild metadata attached to the original XAML file. The `ThemeColumns` element is used to provide a semi-colon separated list of theme aspect names, one for each image in the horizontal strip, in the same order.

Similar metadata is required to specify how the background of an individual image is processed. The `RemoveBackgroundElement` describes this behaviour. To all intents and purposes, there is no need to configure this, other than to set the value to `True`.

Both pieces of metadata are required, or the custom task will not process the XAML file. To add the metadata, edit the `.csproj` project file, find the element that represents the original XAML file and set it as follows:

```xml
<ThemedIconsXamlV3 Include="path-to-xaml-file">
  <ThemeColumns>Gray;GrayDark;Color</ThemeColumns>
  <RemoveBackgroundElement>True</RemoveBackgroundElement>
</ThemedIconsXamlV3>
```

The names of the theme aspects are listed in the [theming section](Theming_Icons.md).

## PNG icons

ReSharper also supports using PNG files as icons, although it is recommended to use XAML instead, as XAML files can scale better to larger sizes and higher screen DPI resolutions.

In order to create an icon from a PNG file, add the file to the Visual Studio project, and set its build action to `ThemedIconPng`. The PNG file will typically be square, e.g. 16×16, although there are no restrictions to enforce this - a larger icon could be included for high DPI or simply for displaying at a larger size.

Theming is supported by a naming convention. The theme aspect name should be placed in square brackets, e.g. `Foo[GrayDark].png`, and a separate PNG file used for each theme aspect. If a theme aspect name isn't specified, the image is treated as a fallback image when failing to find a theme specific image.

The theme aspect names are listed in the [theming section](Theming_Icons.md).

## Generated files

The custom task generates two files - a new XAML file containing the processed images, and a C# file that contains the `CompiledIconId` instances.

The filenames are based on the name of the containing folder, using the format `ThemedIcons.<DIR>.generated.cs` and `ThemedIcons.<DIR>.generated.xaml`, where `<DIR>` is the name of the containing folder, minus any `Icons` suffix. E.g. the folder `BarIcons` would create files called `ThemedIcons.Bar.generated.cs` and `ThemedIcons.Bar.generated.xaml`.

The generated XAML is a `ResourceDictionary` where each themed image is created as an individual `DrawingBrush`, annotated with the name of the icon and the theme aspect it is intended for.

The generated C# file contains `CompiledIconId` instances used to identify and consume the icons. The generated classes are created inside a sealed class to prevent any clashes with other classes inside that namespace. Each class derives from the empty, abstract `CompiledIconClass` class and are marked with the `[CompiledIconClass]` attribute, to allow for discovery of icon XAML files at runtime. The class also declares a couple of assembly level attributes, to help with identifying and loading the icon XAML files, assigning the friendly name to the image in the XAML, and to register a XAML namespace for the icons.

Each class has a single public static field called `Id`, which is the instance of `IconId` that you use when consuming the icon.

```csharp
namespace Foo
{
  // Name based on the folder containing the images - Bar
  public sealed class BarThemedIcons
  {
    // The attribute allows for discovery of the XAML. The first argument
    // is the XAML pack address of the embedded XAML resource. The second
    // argument is the icon index, and the third is the name
    [CompiledIconClass(".../ThemedIcons.Bar.generated.xaml", 0, "Quux")]
    public sealed class Quux : CompiledIconClass
    {
      public static IconId Id = new CompiledIconId(".../ThemedIcons.Bar.generated.xaml", 0, "Quux");
    }
  }
}
```

Once the files have been generated for the first time, they should be added to the Visual Studio project.

## Other useful metadata

The custom task also recognises the `FullPath` metadata, which is useful if the icons are being linked into the Visual Studio project from another location, perhaps shared between projects. The `FullPath` metadata should be the expected full path to the output location, where the generated files will be placed.