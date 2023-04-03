[//]: # (title: Icons)

Icons make it easy to find and explore features in ReSharper and Rider. They can appear in buttons that execute actions, in navigation to differentiate between members, in the gutter to highlight source code, or as various indicators to visualize results (unit testing, solution-wide analysis).

Typically, icons come in different colors and tones to match the user-selected theme (light, dark, etc.). You can change the theme:

<table>
    <tr>
        <td>Product</td>
        <td>Menu Path</td>
    </tr>
    <tr>
        <td>ReSharper</td>
        <td><menupath>Options | Environment | General | User Interface | Application icons theme</menupath></td>
    </tr>
    <tr>
        <td>Rider</td>
        <td><menupath>Preferences | Appearance &amp; Behavior | Appearance | Theme</menupath></td>
    </tr>
    <tr>
        <td>Visual Studio</td>
        <td><menupath>Options | Environment | General | Color Theme</menupath></td>
    </tr>
</table>

[//]: # (## Plugin Icon)
[//]: # ()
[//]: # (- `pluginIcon.svg` and `pluginIcon_dark.svg` under `src/rider/main/resources/META-INF`)

## Themed Icons

The SDK comes with an extensive set of icons that are probably already familiar to you.

You can browse the library in ReSharper's internal mode (`devenv.exe /ReSharper.Internal`) by navigating to <menupath>ReSharper | Internal | Windows | Themed Icon Viewer</menupath>:

![Themed Icon Viewer](Platform__Icons__Themed_Icon_Viewer.png)

> Select _Browse as List_ or _Browse as Icon Packs_ to filter and search icons from the text-box.
>
{type="tip"}

On the right-hand side, you can see the name of the icon as well as the icon pack it belongs to (here `AddMissing` and `CommonThemedIcons`). Those names should be enough to reference them in your code through [code-completion](https://www.jetbrains.com/help/rider/Auto-Completing_Code.html). Each icon is contained in its own class under the `*ThemedIcons` icon pack class. Depending on the way the SDK is asking for an icon, you can pass them as:

- `typeof(MyPluginThemedIcons.Icon)` or
- `MyPluginThemedIcons.Icon.Id`

## Custom Compiled Icons

Similar to the icons that come out-of-the-box, you can add your own icons as _compiled icons_. Although compiled icons can also be generated from PNGs, it is highly recommended to use SVGs, as they'll render without any issues on all resolutions.

### Prepare SVG for Conversion

SVG is a very rich format, but ReSharper can only create compiled icons from so-called _optimized SVGs_. Once you have your SVG, you can optimize it using the [latest version of Inkscape](https://inkscape.org/release/).

Open the SVG, select <menupath>File | Save As...</menupath>, choose <control>Optimized SVG</control>, and enable the following options after hitting <control>Save</control>:

- Options
  - Shorten color values
  - Convert CSS attributes to XML attributes
- SVG Output
  - Remove the XML declaration
  - Remove metadata
  - Remove comments
- IDs
  - Remove unused IDs

> If you're developing on macOS or Linux, feel free to [reach out to us](getting_help.md#problems-with-code), and we will generate the compiled icons for you.
>
{type="note"}

### Prepare for different Themes

If your icons should adapt to different themes, you must provide multiple files with a theme-selector next to each other:

- `<name>[Color].svg` – corresponds to light themes
- `<name>[GrayDark].svg` – corresponds to dark themes

> If your icon is suitable for both, light and dark themes, you don't need to add theme-selectors.
>
{type="note"}

### Export Compiled Icons

Once your SVG icons are prepared and located in a common directory:

1. Open the _Themed Icon Viewer_
2. Choose <control>Add Pane | Directory with Icon Files</control>
3. Select all icons you want to export
4. Choose <control>Export | Export C# Code – SVG Body</control>

A file with the compiled icons should open. You can rename any of the icon classes, the icon pack class, or move them to another namespace.

>  When targeting Rider, you should keep in mind that the pack name is defined through the parent class name of your icon classes with `ThemedIcons` trimmed from the end. If the resulting string is empty, the pack name is `Unnamed`. If you choose to keep your icon classes un-nested, the pack name is `Ungrouped`.
>
{type="warning"}

## Rider Icons

Icons that only surface in Rider (e.g., for run configurations or tool windows) are handled through the IntelliJ SDK. The equivalent of exporting compiled icons is to create a `MyPluginThemedIcons.kt` as follows:

```kotlin
package icons

import com.intellij.openapi.util.IconLoader

// Feel free to rename
object MyPluginThemedIcons {
    @JvmField val Icon = IconLoader.getIcon("/resharper/MyPlugin/Icon.svg", javaClass)
}
```

The `MyPluginThemedIcons.kt` and your SVG icons should be located as follows:

```text
src/rider/main
├── kotlin/icons
│   └── MyPluginThemedIcons.kt
└── resources
    └── resharper
        └── MyPlugin           # pack name with 'ThemedIcons' trimmed
            ├── <Icon>.svg
            └── <Icon_Dark.svg
```

> Please refer to the [IntelliJ Platform SDK](https://plugins.jetbrains.com/docs/intellij/work-with-icons-and-images.html) for more information.
>
{type="note"}
