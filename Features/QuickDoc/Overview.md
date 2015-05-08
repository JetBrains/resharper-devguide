---
---

# QuickDoc

The QuickDoc feature displays a popup window with details of the code element at the current text caret position. What gets displayed depends on what that code element is.

For example, a C# QuickDoc window will show the element name and type, with hyperlinks to navigate to any type definitions and description from XML documentation. 

A CSS QuickDoc window will show the expected values for the property, with hyperlinks to the definitions of the value units, and a "read more" link to see the W3C website's documentation.

<!-- Insert picture -->

ReSharper ships with full support for C# and VB, compiled CLR types, JavaScript, CSS, XML and HTML out of the box, but it is possible to extend QuickDoc to provide documentation for languages or code elements not supported by default.

## Read more

1. Implement your own support, by implementing [IQuickDocProvider](ImplementingProvider.md) and [IQuickDocPresenter](ImplementingPresenter.md), using the [helper classes](HelperClasses.md)
2. [Extend the existing providers](ExistingProviders.md)
3. [Test QuickDoc providers](Testing.md)
