---
title: Gold Files
---

ReSharper's test framework typically takes an input file, adds it as source file to an in-memory project, runs a process on it, such as inspections or code completion, and records the output in a temporary text file. This `.tmp` file is then compared to a `.gold` file, which contains the expected output. If they match, the test passes. If they don't, the test fails.

The input file typically lives in the same folder under the `test\data` folder as the `.gold` file. The input file and the gold file should both be committed to source control. The `.tmp` file should not be committed, as it represents a failing test.

The contents of the gold file depends very much on the test. For example, gold files for code completion tests will be a list of items in the code completion list, while inspections will be a copy of the source file annotated with location and text of highlights. Details of the contents of gold files can be found throughout this guide, in the specific sections on testing a particular feature.

## Input files

The input file typically lives in the same `test\data` sub-folder as the `.gold` file. Exactly what the input file is depends on the test, but mostly, it is a source file, such as `.cs`, `.js`, `.html`, etc. However, it can also be a compiled assembly, i.e. a `.dll` file.

Depending on the type of the file, it can be handled in different ways. It can be added as a source file to the in-memory project that the test environment creates, and the test will run some process over that file, such as running inspections, or invoking code completion. An assembly could be added as a reference to the in-memory project, and the test might perform metadata analysis (like Reflection) or decompilation, and put the output of that process to the `.tmp` file.

## Special tokens

> **TODO** Document the default special tokens

## Creating gold files

To initially make a test pass, a known good gold file is required. The easiest way to create this is to run the test without a `.gold` file - which will cause it to fail. The resulting `.tmp` file should then be manually examined to see if it is the correct output for the test. If it is, it should be renamed to `.gold`.

## Working with failures

When a test fails, ReSharper's test framework adds extra information into the [`Data` dictionary of the exception](http://msdn.microsoft.com/en-us/library/system.exception.data.aspx) that the failing test throws. This data provides extra context about the test, and is rendered as part of the test failure message in the unit test runner. The following fields are added:

| Field name       | Contents
|------------------|---------
| `GoldFile`       | The path to the `.gold` file. This is rendered as a clickable `file://` url that will open Windows Explorer with the `.gold` file selected.
| `TempFile`       | The path to the `.tmp` file. This is rendered as a clickable `file://` url that will open Windows Explorer with the `.tmp` file selected.
| `Cmd`            | A clickable `file://` url that will open a command prompt to the `test\data` sub-folder that contains the `.gold` or `.tmp` file.
| `Directory`      | A plain text string that contains the sub-folder under `test\data` that contains the `.gold` or `.tmp` file.
| `Diff`           | A diff between the `.gold` and `.tmp` files.
| `CopyToGold`     | A clickable `file://` url that will execute a `.bat` file that's been created in the `%TEMP%` folder. The batch file will copy the `.tmp` file to the `.gold` file. This field is added when the `.gold` file doesn't already exist.
| `UpdateGold`     | A clickable `file://` url that will execute a `.bat` file that's been created in the `%TEMP%` folder. The batch file will copy the `.tmp` file over an existing `.gold` file. This should be used only once the differences between the existing `.gold` and the .`tmp` file have been examined and found to be correct.
| `ComparisonFile`&nbsp;&nbsp;&nbsp; | A clickable `file://` url that will execute a `.bat` file that's been created in the `%TEMP%` folder. This batch file will open a visual diff tool to show the differences between the `.tmp` file and the `.gold` file. See below for more details.
| `GoldLine`       | Can contain the first line of the `.gold` file that does not match against a new `.tmp` file. This is only used by certain tests.
| `TestLine`       | Can contain the first line of the `.tmp` file that does not match against a `.gold` file. This is only used by certain tests.
| `LineNumber`     | Can contain the line number of the first line in both the `.tmp` and `.gold` file that does not match. This is only used by certain tests.

> **NOTE** Failures create `.bat` files in the `%TEMP%` folder that will update the `.gold` file, or run a visual diff tool. These `.bat` files are not cleaned up automatically (as the test run has already finished and the `.bat` files are used when browsing results). The `%TEMP%` folder should be manually cleaned up from time to time.

## Visual diff tools

When a test fails, the `ComparisonFile` field is added to the failing test's exception's `Data` dictionary. This field contains a clickable `file://` url that will execute a `.bat` file that's been saved in the `%TEMP%` folder. This batch file is as follows:

```dosbatch
setlocal
if not defined DIFF SET DIFF=kdiff3
if not defined DIFF_PARAMETERS SET DIFF_PARAMETERS= --L1 '$plabel1' --L2 '$clabel' $parent $child
set DIFF_PARAMETERS=%DIFF_PARAMETERS:$plabel1=Input.cs.tmp%
set DIFF_PARAMETERS=%DIFF_PARAMETERS:$clabel=Input.cs.gold%
set DIFF_PARAMETERS=%DIFF_PARAMETERS:$parent="C:\Users\matt\Code\MyPlugin\test\data\Completion\Input.cs.tmp"%
set DIFF_PARAMETERS=%DIFF_PARAMETERS:$child="C:\Users\matt\Code\MyPlugin\test\data\Completion\Input.cs.gold"%
start /B "" "%DIFF%" %DIFF_PARAMETERS%
endlocal
```

By default, it will launch the [KDiff3](http://kdiff3.sourceforge.net) diff tool, passing in the parameters to display the `.tmp` and `.gold` files side by side. This can be overridden by defining the `%DIFF%` and `%DIFF_PARAMETERS%` environment variables.

The `%DIFF%` environment variable should be defined to the command needed to start the diff tool. This can be a fully qualified path, or just the executable name if it's already on the path.

The `%DIFF_PARAMETER%` environment variable specifies the command line for the diff tool. The batch file will substitute the `.temp` and `.gold` names and full file paths for specific placeholders:

| Placeholder | Value
|-------------|------
| `$plabel1`  | File name of the `.tmp` file.
| `$clabel`   | File name of the `.gold`&nbsp;file.
| `$parent`   | Full path to the `.tmp` file.
| `$child`    | Full path to the `.gold` file.

For example, to use [SourceGear DiffMerge](https://sourcegear.com/diffmerge/) as the diff tool, you can set:

| Environment&nbsp;variable&nbsp;&nbsp;&nbsp;| Value                                         |
|----------------------|-----------------------------------------------|
| `DIFF`               | `sgdm`                                        |
| `DIFF_PARAMETERS`    | `/t1="$plabel1" /t2="$clabel" $parent $child` |

