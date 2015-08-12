---
redirect_from:
  - /Plugins/Testing/OptionsIterator.html
---

# Combinatorial Testing

The test framework supports running a test multiple times using different product settings. The settings can be specified in a text file, via an attribute or via a parameter embedded in the test input file. The base class for the test must support this explicitly - it is not available to all tests by default.

> **NOTE** This functionality serves a different purpose to NUnit's `[TestCase]` and `[TestCaseSource]` attributes, which are also used frequently across the Platform. The purpose of NUnit's attributes is to run a test method multiple times with different parameters passed in. The purpose of ReSharper's combinatorial testing is to run a test method multiple times, changing the environment (settings) each time it's run. It does this within a single test method run - only one test is executed, and all results are stored in the same file.

* Table of Contents
{:toc}

## Overview

The base class uses the `TestOptionsIterator.Iterate` method to run a lambda multiple times, temporarily changing settings before each run. Typically, it's called as part of `ExecuteWithGold`, and passes a `TextWriter` to the lambda to write to a `.tmp` file ready to be compared to a `.gold` file.

```csharp
ExecuteWithGold(inputFile, writer =>
  myTestOptionsIterator.Iterate(writer, settingsStore, item, Solution, (data, writer1) =>
  {
    // Run the test, output to writer1
  })
);
```

The instance of `TestOptionsIterator` is usually created in the constructor:

```csharp
public MyTest()
{
  myTestOptionsIterator = new TestOptionsIterator(this)
  {
    ReportUnaffected = false;
  };
}
```

When running, the options iterator will run the test multiple times, once for each combination of configured settings. The results are all written to the same `.tmp` file, separated by a marker line, and listing details of the settings that were valid for that run. Once the run is complete, the `.tmp` file is compared against the `.gold` file. If there are any differences, the test fails.

The `ReportUnaffected` property controls how the runs are output. If the value is set to `true`, then each combination of settings is output. If set to `false`, only combinations of settings that produce a previously unseen value are output. This is normally handled by the base class of the item you're trying to test, and is not usually available to change.

## Supported test base classes

The use of `TestOptionsIterator` is opt-in, and only supported by a number of test base classes. The following are the list of currently supported base classes.

* `CodeCleanupTestBase`
* `HighlightingTestBase`
* `TypingAssistTestBase`
* `CodeFormatterWithExplicitSettingsTestBase`
* `CSharpQuickFixTestBase`
* `TypeScriptQuickFixTestBase`

Of course, the class is available to use in your own test code, too.

## Describing combinations

Combinations of options are specified with a JavaScript-like DSL. A JavaScript expression describes a set of options and how they are to be combined.

At the simplest, an object literal is used as a collection of name/value pairs to set options and their values.

```js
{
  TagAttributesFormat: OnSingleLine,
  MaxSingleLineTagLength: 10
}
```

Values in the object literal can be string literals, numerals, or unquoted values (this is JavaScript-like, not JSON). Enum values, booleans and numerals are automatically parsed and converted to the appropriate type.

In order to support combinations of options, the value of the property can be an array literal. The array lists all of the values that should be applied to that property.

```js
{
  TagAttributesFormat: [OnSingleLine, OnDifferentLines],
  MaxSingleLineTagLength: 10
}
```

Given this configuration, the test will run twice, once for `OnSingleLine, 10` and again for `OnDifferentLines, 10`. Given another property with an array literal, the test will be run for all combinations of all values. Adding more properties, with more options, can very quickly increase the number of times the test is run.

Instead of creating an array literal with all enum values listed, the object literal property can use the special value `"all"` or the wildcard `"*"`, which when applied to an enum option will run a combination of all values for that option property.

### Property names

In order to know what options the property names map to, the test class and its base classes can use the `[TestSettingsKey]` attribute to specify one or more settings key, as used by ReSharper's settings system.

```csharp
[TestSettingsKey(typeof(HtmlFormatterSettingsKey))]
public class HtmlCodeFormatterTests : CodeFormatterWithExplicitSettingsTestBase
{
  // ...
}

[TestSettingsKey(typeof(CommonFormatterSettingsKey))]
public class CodeFormatterWithExplicitSettingsTestBase // : ...
{
  // ...
}
```

The `HtmlCodeFormatterTests` can now use properties from `HtmlFormatterSettingsKey` and `CommonFormatterSettingsKey` in the object literals of the DSL. It can use each property name unqualified, or disambiguate by using the friendly name of the settings key as a qualifier:

```js
{
  TagAttributesFormat: OnSingleLine, // Sets HtmlCodeFormatterTests.TagAttributesFormat
  CommonFormatting$ALIGNMENT_TAB_FILL_STYLE: USE_SPACES // Sets CommonFormatterSettingsKey.ALIGNMENT_TAB_FILL_STYLE
}
```

### Special property names

The `TestOptionsIterator` recognises a couple of special property names for settings that are usually controlled by Visual Studio rather than ReSharper:

* `USE_TABS` is a boolean value to use tabs rather than spaces.
* `INDENT_SIZE` is an integer value to specify the size of an indent.

Both of these values are language specific, so require a language prefix:

```js
{
  HTML$USE_TABS: all
}
```

This example will run the test code multiple times, with the "use tabs" settings for HTML files set to `true` and `false`.

The language name is the name of the language's `PsiLanguageType` instance, e.g. `HtmlLanguage.Name`, `CssLanguage.Name`, etc.

### Custom property names

Tests can also iterate over values not stored in the settings subsystem. Rather than automatically setting the option in settings before running the test, the value is made available to the test in the `CustomValues` dictionary of the test data passed to the test function.

```csharp
ExecuteWithGold(inputFile, writer =>
  myTestOptionsIterator.Iterate(writer, settingsStore, item, Solution, (data, writer1) =>
  {
    var customValue = data["myCustomVariable"] as CustomType;
    // Run the test using customValue, output to writer1
  })
);
```

The test class needs to tell the `TestOptionsIterator` that a custom value can be used, by decorating the class (or base class) with the `[TestSettingsVariable]` attribute. It must pass in the name and `Type` of the variable, which can be either boolean or an enum.

```csharp
[TestSettingsVariable("FormatProfile", typeof(CodeFormatProfile))]
public abstract class CodeFormatterWithExplicitSettingsTestBase // : ...
{
  // ...
}

public enum CodeFormatProfile
{
  DEFAULT,
  INDENT,
  GENERATOR,
  SOFT,
}
```

The DSL can now use `"FormatProfile"` as a property name, and provide values either as a single value, array literal or the wildcard `"*"` (the `"all"` value doesn't work here). The current value will be available by calling `data.CustomValues["FormatProfile"]`.

## Advanced combinations

The DSL can handle more constructs than just object literals. It supports:

### Object literals

Used to provide name/value pairs to set options. The value can be a scalar value (such as an integer) or a collection of values.

If the value is a scalar, it can be a string literal, numeral, boolean or unquoted reference, in which case it's treated as an unquoted string literal. The value is converted to the appropriate type for the setting - string, integer, boolean or enum. The magic values `"all"` or `"*"` are wildcards and create a combination of all of the values for the type - `true` and `false` for booleans, or all values for an enum.

To represent a collection of items, the value can be either an array literal or a nested object literal. An array literal creates a combination of all of the simple (quoted/unquoted) string values in the array. An object literal is used to create string name/value pairs to be used for an indexed setting.

```js
{
  FileExtensions: [ "*.html", "*.cshtml" ],
  TemplateMimeTypes: {
    "text/html": true,
    "text/plain": false
  }
}
```

This will create a combination of tests for the two values in `FileExtensions`, and an indexed setting for `TemplateMimeTypes` with the two keys `"text/html"` and `"text/plain"` to `true` and `false`, respectively.

If a property's value is an array literal, it can further contain either an object literal, or even another array literal. This allows for creating a combination of indexed settings. 

```js
{
  FileExtensions: [ [ "*.html", "*.cshtml" ], [ "*.html", "*.vbhtml" ] ],
  TemplateMimeTypes: [
    { "text/html": true, "text/plain": true },
    { "text/html": false, "text/plain": false }
  ]
}
```

This creates a set of combinations of indexed values:

* `*.html` and `*.cshtml` is set, and the mime types are set to true
* `*.html` and `*.cshtml` is set, and the mime types are set to false
* `*.html` and `*.vbhtml` is set, and the mime types are set to true
* `*.html` and `*.vbhtml` is set, and the mime types are set to false

### Array literals

When used inside an object literal, the behaviour is as described above.

Otherwise, array literals are used to create a sequential combination of its contents. That is, each item in the array literal is evaluated and the test is run. The item in the array literal must either be another array literal (which creates a new combination grouping), or an object literal, which describes the settings to apply, as above.

```js
[
  { Reformat: [ True, False ] },
  { HTML$USE_TABS: [ True, False] }
]
```

This runs the test four times, firstly with both boolean values for `Reformat` and secondly with both boolean values of "use tabs" for HTML files. Note that it doesn't combine between the two - the values of "use tabs" are not modified during the two runs when `Reformat` is modified.

### Binary expressions

Binary expressions are used to combine combinations:

```js
{ Reformat: all } * { HTML$USE_TABS: all }
```

This example will combine each value of the "use tabs" settings with each value of the `Reformat` setting.

### Reference expressions and parentheses

A simple JavaScript-like reference to a name is used to refer to common, shared combination definitions, as discussed below.

The DSL understands parentheses, and safely strips them. E.g. `"({SPACES: all} * {ALIGN: all})"` is safely handled as a combination of setting `SPACES` and `ALIGN`.

## Using combinations

There are several ways to apply the combinations to a test. The simplest way is to use the `[TestSettings]` attribute to specify one or more combinations directly on the test class or base class. Alternatively, the combinations can come from a file that live side-by-side with the input path, or can be written, inline, into the input path.

If more than one method is used, the combinations are combined (as a binary expression, see the advanced combinations section above).

### Combinations as attributes

The `[TestSettings]` attribute can be used multiple times on a method, class or base class to provide JavaScript-like expressions to represent combinations to apply to the test.

```csharp
[TestSettings("{ TagAttributeFormat: all }")]
[Test]
public void Test001()
{
  DoNamedTest();
}
```

### JCNF files

The `TestOptionsIterator` will automatically load combinations specified in a `.jcnf` file named after the project file being tested. For example, if the test is being run on `test01.js`, the `TestOptionsIterator` will try to load a file called `test01.js.jcnf`.

The file should contain a set of combinations as a single object literal:

```js
{
  TagAttributesFormat: OnSingleLine,
  MaxSingleLineTagLength: 10
}
```

If the `TestOptionsIterator` sees a `.dotSettings` file (e.g. `test01.js.dotSettings`) it will automatically convert it to a `.jcnf` file. The resulting file will not contain any combinations - each setting will only have one value - but it will give you a good place to start.

### Embedded combinations

Combinations can also be embedded directly in the input file, using the `SettingsIterator` variable:

```js
${SettingsIterator: { TagAttributeFormat: all } }
function foo() {}
```

### Sharing combinations

Common combinations can be described in separate files, and shared and included in multiple tests. The test class, or base class needs to use one or more `[TestSettingsInclude]` attributes to point to a file to be loaded.

```csharp
[TestSettingsInclude(@"codeInsight\CodeFormatter\JavaScript\common.jcnf")]
public abstract class JavaScriptCodeFormatterTestBase : CodeFormatterWithExplicitSettingsTestBase
{
  // ...
}
```

The file is a `.jcnf` file as described above, however the format is slightly different. Instead of describing a single JavaScript-like object literal, the common `.jcnf` files contain JavaScript-like variable declarations that refer to array literals that describe a combination:

```js
var Align = [
  "Align",
  { ALIGN_MULTILINE_PARAMETER : all },
  { ALIGN_MULTIPLE_DECLARATION : all },
  { ALIGN_TERNARY : all }  
];
```

The syntax used here is as described in the [advanced combinations](#advanced-combinations) section above. The first element is the name of the combination, displayed when the combination needs to be output. This example will run multiple times, for every combination of each value of each property (two boolean properties plus an enum value with three values gives twelve combinations).

The variable name is available to be used by the main DSL, as a simple reference to the name:

```csharp
[TestSettings("Align")]
[Test]
public void Test001()
{
  DoNamedTest();
}
```

Or it can be combined into a binary expression:

```js
( Align * { SPACE_AROUND_BINARY_OPERATOR: all } )
```

Which will expand the combinations by also iterating over all values of `SPACE_AROUND_BINARY_OPERATOR` (which is a boolean).

