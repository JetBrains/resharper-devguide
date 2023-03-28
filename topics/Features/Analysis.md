[//]: # (title: Analysis)

ReSharper analyses your code on-the-fly, as you type. It has several modes of analysis, but the output is always the same - a list of "highlights". ReSharper supports several types of highlights - some of them change the colour of existing text, while others draw new content onto the editor, such as wavy underlines.

ReSharper supports:

* **Squigglies** - error, warning and suggestion wavy underlines. Also, hints are shown as a short series of dots underlining the start of an identifier.
* **Identifiers** - ReSharper can change the colour of identifiers based on their semantics, such as parameter, field or variable. It can also use a bold font, e.g. for mutable variables.
* **Colour highlighting** - a solid underline underneath usages of colours in C# and CSS, indicating what colour is specified by the given constant parameters.
* **Gutter icons** - ReSharper can show an icon in the selection gutter of the text editor, to indicate e.g. overridden methods or recursive method calls. These gutter icons are another form of highlight, and can include bulb items in the same way that Quick Fixes and Context Actions are displayed when the user clicks the icon, or uses <kbd>Alt</kbd>+<kbd>Enter</kbd> on that line.
* **Custom highlights** - analysis can add custom highlights, with a number of different effects, such as a solid colour underline, which can indicate additional functionality. For example, ASP.NET MVC support highlights string literals as MVC actions with a solid underline, indicating it can be clicked on for navigation. Similarly, the HeapView plugin uses a solid underline to indicate code that causes memory allocations. Other effects include changing the text background, text itself, outlines, underlines and gutter icons.

Typically, extensions are most interested in the "squigglies" - adding analysis to add warning or suggestion highlights to the editor. ReSharper provides several ways of doing this, such as "daemon stages" and Element Problem Analysers. These highlights can be static, or configurable. A static highlight is hard coded to be a certain severity, such as a compiler error, while configurable highlights can have their default severity changed by the user. These highlights need to be registered with ReSharper, in the form of assembly attributes.

ReSharper also supports suppressing these inspections with specially formatted comments.
