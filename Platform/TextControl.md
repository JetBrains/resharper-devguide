# TextControl

Though for the most part plug-ins work by manipulating the AST, it is sometimes necessary to interact with the underlying text control. One typical example of this is when deploying _highlightings_ - those wavy lines that are used to underline code that needs to be corrected.

Interaction with the text control is done via the _TextControl abstraction_. This abstraction, represented by an `ITextControl` and related interfaces, abstracts away the physical text editing control by putting a View on top of this. There are currently two varieties of such a view: one supporting Visual Studio and another supporting dotPeek.

## ITextControl

The `ITextControl` interface is a view for editing an `IDocument`. It contains, among other things, information about the position of the caret and selection, the text control’s scrolling parameters, and of course exposes an `IDocument`, i.e. the document that the text control supplies.

In case you want to show code-related UI elements, the text control also provides an interface for coordinate conversion. There is also an `IsPositionInTextView()` method which can tell you whether a particular text control position is in the view area of the text control. You have probably see this in action in terms of bulb actions and blue pop-up windows that only show up in relation to the code that’s currently visible on screen.

## TextRange and Offsets

The `TextRange` type is used to store information about a _range_, i.e. a selection of characters starting and ending at particular positions (offsets). Direct textual manipulation of a document typically happens by way of either `TextRange` parameters or integer offsets. For example, you can use `IDocument.DeleteText` to delete text at the specified `TextRange`.

In certain cases, the document itself exposes information about offsets. For example, if you want to get a `TextRange` for a whole line, you would construct a `TextRange` from the values provided by `IDocument.GetLineStartOffset()` and `IDocument.GetLineEndOffsetNoLineBreak()`. Both of these methods take the line number as a parameter.

`TextRange` contains a number of utility methods for working with ranges such as, e.g., creating an intersection of two ranges. Some of these methods are also mirrored in the API of the `DocumentRange` type (see below).

## DocumentRange

The `DocumentRange` type is simply a container for the `TextRange` as well as the document it relates to. This type is used in, e.g., the construction of highlightings.

