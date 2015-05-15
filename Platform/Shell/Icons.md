---
---

# Icons

ReSharper supports icons in several places, such as options pages, gutter icons, tied to an action for context menus or toolbars for tool windows. Complicating matters is that ReSharper needs to match the host platform, more specifically, icons need to match the current Visual Studio theme. Also, ReSharper needs to provide support for high DPI scenarios, either by providing multiple bitmap images, automatically scaling, or supporting vector images.

In order to work with these requirements, ReSharper provides an API for consuming icons, rather than working directly with images. Each icon is identified by an icon id, which is a class that derives from `IconId`. Given an `IconId`, ReSharper can create either a `System.Drawing` (Windows Forms) compatible bitmap, scaled to the current screen DPI, or a WPF `ImageSource`, which can be either a bitmap or a scalable vector image. Furthermore, it can return a "live" version of the icon, which uses `IProperty<T>` to wrap the icon and provide notifications when it's updated, such as when a theme changes.

ReSharper provides several different types of `IconId`. The most common is `CompiledIconId`, which is a reference to an icon that has been compiled into an assembly's resources. These icons can be created either from PNG files, or scalable WPF `DrawingBrush`. Other types include animated icons, effects applied to icons (such as alpha) and filetype icons.

Themed icons are supported by the icon ID owners, the components that use the derived `IconId` instances to load the images. When appropriate, they combine the icon ID with the current theme aspect names (e.g. "Dark", "Light", "Color", "SymbolsVs11Color", etc.).

Internally, icons are represented by WPF `ImageSource` instances, which means transparency is understood and honoured.
