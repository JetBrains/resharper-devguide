# Live Template macros

Of course, plugins can provide macros to extend the list already included with ReSharper (and ReSharper extensions can include Live Templates that refer to these templates, either by bundling the macro plugin into the same extension, or taking a dependency on another extension that contains a macro plugin. See the Packaging page for more details).

Macros are implemented with two classes - a definition, and an implementation. The definition describes the macro, and the implementation handles the macro being invoked.

## Macro Definition

The macro definition is a class that implements `IMacroDefinition` and is marked with the `MacroDefinitionAttribute`. You can derive from `SimpleMacroDefinition`, which provides a default implementation of the interface.

The attribute derives from the Component Model's `ShellComponentAttribute`, which means the definition class is instantiated once when the Shell starts up, and essentially lasts for the length of the lifetime of the application. It consists of three string properties - `Name`, `ShortDescription` and `LongDescription`. The name is a simple identifier for the macro definition, and can be passed to `MacroManager.GetMacroDefinition` if you need to retrieve a macro definition programmatically. The short description is displayed in the list of macros the template editor shows, and the long description is displayed when the macro definition is selected in this list.

The short description can also contain placeholders that indicate that a macro can be parameterised. This is only for display purposes - parameters are declared by the macro definition class, described below. When the short description is shown in the list of macros in the template editor, the placeholder text is shown in bold, indicating to the user that the macro can be parameterised, and what part is parameterised.

The format of the placeholder is `{X:name}`, where the `X` is replaced by the parameter index, and the "name" is the text that is displayed in bold. (Strictly speaking, the "name" text is the name of the parameter, which is why there is also a parameter index. The name is passed to the parameter value UI, where it could be used e.g. as a label, however, it's not currently used.) For one variable, you can have a description such as "Insert reference to {0:type}". This states that the macro will take one parameter, called "type", and the list of macros shown in the template editor will display "type" in bold. Multiple parameters can be referenced by incrementing the parameter index, for example: "Delimited {0:list} of items with {1:delimiter}" could be a macro that uses the first parameter "list" to store a delimited list of string values, using the delimiter specified in the parameter called "delimiter" at index 1.

The `IMacroDefinition` interface has two members - a `Parameters` property and a `GetPlaceholder` method.

```cs
public interface IMacroDefinition
{
  ParameterInfo[] Parameters { get; }
  string GetPlaceholder(IDocument document, IEnumerable<IMacroParameterValue> parameters);
}
```

The `Parameters` property returns an array of `ParameterInfo` classes that describe the type of the parameters required by the macro. The class is essentially holds an enum, declaring the parameter as either a string, a type, or a reference to another variable in the template. They are used to provide different UI when entering parameter values in the template editor. The `SimpleMacroDefinition` implementation of `Parameters` returns an empty array, and should be overridden if required.

The `GetPlaceholder` method is called when the template is first expanded and before the user has a chance to edit any hotspots. Each hotspot's macro definition is queried for its placeholder value, and the value is substituted into the expanded text. The expanded text is then formatted, according to the current document's language. In other words, the purpose of the placeholder value is to provide an appropriate value that allows the resulting text snippet to be accurately parsed and formatted. For example, a macro that is expected to return an identifier (e.g. to populate a variable name) could return a simple placeholder of "a", which is a valid identifier. However, it shouldn't return something like "a - b - c", which is an expression, and if used as an identifier for a variable name, would cause a parse error. The default implementation in `SimpleMacroDefinition` returns the text "a", but this can be overridden in a derived class.

## Macro Implementation

The macro's implementation is handled by a class that implements `IMacroImplementation` and is marked with the `MacroImplementationAttribute`. In a similar manner to the macro definition, the `SimpleMacroImplementation` base class provides a good default implementation to derive from.

The `MacroImplementationAttribute` defines several members:

* `Definition` returns the `System.Type` of the `IMacroDefinition` class that is being implemented. For example, the `GuidMacroImpl` class that implements the macro for the "nguid" template refers to `typeof(GuidMacroDef)`.
* If the `IsExpandAndSkip` property is set to true, this effectively makes the hotspot read only. The value is expanded and not changed, and you can no longer tab to the hotspot. The effect is the same as if the variable in the template is marked as not editable.
* The `ScopeProvider` and `GetScopes` members allows a macro implementation to state which scopes it supports. This allows you to limit an implementation to files of a specific languages or even locations within these files. The `MacroImplementationAttribute` defaults this value to set the scope to be "everywhere", meaning the macro implementation is not limited in where it can be used. In order to override this behaviour, you would need to derive from `MacroImplementationAttribute` and override the `ScopeProvider` property to return a new type that implements `IMacroImplementationScopeProvider`.

Multiple implementations can be provided for a single definition, and ReSharper will attempt to find the implementation that has the closest scope, as returned by `GetScopes`, to the actual scope in use. For example, a scope of "C# file" will be closer than a scope of "everywhere". If there are multiple implementations, but there is no closest scope, an arbitrary implementation is chosen.

An instance of the implementation class is created for each hotspot. The implementation class can have two constructor parameters injected when the class is created - `IHotspotSession` and `MacroParameterValueCollection`. This is how a parameterised implementation gets hold of the parameter values specified in the Live Template. The constructor parameters can be in any order, or not specified at all. However, they must be marked with the `[Optional]` attribute, because another instance of the implementation is created at shell startup time (`MacroImplementationAttribute` derives from `ShellCompomentAttribute`) and neither the session nor the parameter collections are available at that time. This instance will receive `null` for those parameters. The additional instance is created to map between implementations and definitions, especially when added dynamically from a plugin.

`IMacroImplementation` defines three methods:

```cs
public interface IMacroImplementation
{
  bool HandleExpansion(IHotspotContext context);
  HotspotItems GetLookupItems(IHotspotContext context);
  string EvaluateQuickResult(IHotspotContext context);
}
```

When the template is first expanded, and after the text has been replaced by placeholders and reformatted, each hotspot's **`GetLookupItems`** method is called. If it returns any values, the first is used to replace the placeholder. If no items are returned, the hotspot's **`EvaluateQuickResult`** is called, and that text is used instead.

When hitting `tab` or `shift+tab` to move between the hotspots, each hotspot is activated, and the `HandleExpansion` method is called. This gives the macro implementation the opportunity to handle what happens when the hotspot becomes active, overriding the normal behaviour of showing a completion list of items. For example, the Basic, Smart and Type completion macros invoke the standard code completion components, rather than supplying a list of items to the hotspot manager to display. **`HandleExpansion`** should return `true` if it handles the expansion request, or `false` otherwise.

The `GetLookupItems` method can also get called when a hotspot becomes active (so if you take control in `HandleExpansion`, you should also return `null` or an empty list here). The returned items are cached, and will be reused if the text hasn't been changed, so tabbing between multiple hotspots doesn't cause multiple calls. If the surrounding text changes, the cached items are thrown away, and new items are retrieved, allowing for macros that depend on the surrounding text, such as suggesting names for a variable of a type that can also be edited.

`GetLookupItems` returns an instance of `HotspotItems`, which maintains a list of `ILookupItem` instances, as used in code completion. Generally speaking, this will be a list of `TextLookupItem` instances, to expand to a simple text string. However, you can use any `ILookupItem` instance; the `ILookupItem.Accept` method is called to insert the text. For example, the macro to insert a reference to a type derives from `TypeLookupItem` and overrides a method called from its `Accept` method in order to properly position the text caret for generic types. You can also use the `HotspotItems.Empty` instance if you don't have any lookup items. The list is created once, and doesn't change as you type, although the values are filtered to match what you are typing.

`EvaluateQuickResult` is called when another hotspot changes. It allows for a macro to be based on the contents of other hotspots, or the surrounding text. If the macro is based on a parameter, such as a constant value from the template editor, or a reference to another hotspot variable, the content can be retreived from the `MacroParameterValueCollection` passed in to the constructor. To get the value would look something like this:

```cs
[MacroImplementation(Definition = typeof(ToUpperMacroDef))]
public class ToUpperMacroImpl : SimpleMacroImplementation
{
  private IMacroParameterValueNew myArgument;

  public ToUpperMacroImpl([Optional] MacroParameterValueCollection arguments) {
    myArgument = arguments.OptionalFirstOrDefault();
  }

  public override string EvaluateQuickResult(IHotspotContext context) {
    return myArgument == null ? null : myArgument.GetValue().ToUpperInvariant();
  }
}
```

As the name suggests, `EvaluateQuickResult` needs to be quick, as each macro's implementation is called for every character typed in the current hotspot. Processing should be kept to a minimum.

## IHotspotContext

All of the methods in `IMacroImplementation` take `IHotspotContext` as an argument. This interface has three properties:

```cs
public interface IHotspotContext
{
  IHotspotSession HotspotSession { get; }
  IHotspotSessionContext SessionContext { get; }
  DocumentRange ExpressionRange { get; }
}
```

* **`IHotspotSession HotspotSession`** - provides information about the current session, including methods such as `GoToNextHotspot` and `EndSession`, `GetVariableResult` to get the value of a variable by name (as listed in the macro definition's short description). You can also get the list of hotspots and the current hotspot (which might not be the hotspot associated with the current macro implementation, especially in the call to `EvaluateQuickResult`).
* **`IHotspotSessionContext SessionContext`** - the context of where the template is being instantiated, including the solution, documents and offsets.
* **`DocumentRange ExpressionRange`** - the document range of the current hotspot (again, not necessarily the hotspot for the current macro implementation). You can call `ExpressionRange.GetText()` to retrieve the value of that hotspot, or `HotspotSession.CurrentHotspot.CurrentValue`.

## Helper methods

ReSharper provides a couple of useful classes for implementing macros, namely `MacroUtil` and the `IMacroUtil` interface. Note that these classes aren't related! The `MacroUtil` class is a static class with a couple of useful methods:

* **`SimpleEvaluateResult`** creates an instance of `HotspotItems` from a single string value. Useful for implementing `IMacroImplementation.EvaluateResult`.
* **`GetLanguageType`** and **`GetFile`** return information from an instance of `IHotspotContext`
* **`GetMacroUtil`** returns an instance of `IMacroUtil`

The `IMacroUtil` interface is implemented per language - there is an instance for C#, VB, JS, etc. It contains several methods, that make it easy to e.g. suggest variable types, get the variables that are currently visible, convert a type name into an instance of `IType`, etc. It's primary purpose is to support several of the built in macros, so they can be implemented once, and each language can add support for the more language specific parts of the implementation.

