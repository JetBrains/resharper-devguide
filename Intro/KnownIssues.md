---
---

# Known Issues

> **WARNING** This topic relates to ReSharper 8, and has not been updated to ReSharper 9 or the ReSharper Platform.

This page lists known issues when building plugins.

* Table of Contents
{:toc}

## Plugin keyboard shortcuts don't work

**Problem:** If a plugin defines actions that include keyboard shortcuts, the shortcut is only valid the first time Visual Studio is loaded. The next time Visual Studio restarts, the shortcut is gone. Any customisation of the keyboard shortcut in Visual Studio's options dialog is also lost on restart.

**Solution:** There is no workaround for this issue. It will be addressed in ReSharper 9.0.

**YouTrack:** [RSRP-246461](http://youtrack.jetbrains.com/issue/RSRP-246461), [RSRP-337038](http://youtrack.jetbrains.com/issue/RSRP-337038)

**Details:** ReSharper's built in actions are statically registered as Visual Studio commands at install time, while plugin actions are registered as dynamic commands, every time Visual Studio starts.

When applying key bindings to the Visual Studio commands, either on initial load, or when re-applying a shortcut scheme from the options dialog, ReSharper loops over all known actions, applies any key bindings and sets a flag for that action ID in a settings file. On next load ReSharper verifies that all actions have had their key bindings applied by consulting this list of flags. Any that are missing a flag are new actions, presumably from a plugin, and the shortcut is applied and the flag is set and saved to the settings file.

However, because plugin actions are dynamic, the Visual Studio command is removed when Visual Studio closes. This means any shortcut customisation is lost. On next restart, the flags for these dynamic actions are already set, so the default shortcut isn't applied again.

ReSharper 9.0 plans on addressing this issue by registering all actions statically. The shortcut will only need to be set once, and customisations aren't lost between Visual Studio restarts.

(The location of the settings file with the list of flags is `%LOCALAPPDATA%\JetBrains\ReSharper\vAny\vs12.0\vsActionManager.dotSettings` where `vs12.0` is the particular version and custom hive of the Visual Studio instance.)

## XML files are not processed as part of SWEA

**Problem:** XML files are not processed as part of Solution Wide Error Analysis, so any custom problem analysers for will not show up in SWEA. Build scripts and DTD files are also affected. Resx, web.config and XAML files are processed by SWEA.

**Solution:** There is no workaround for this issue. It will be addressed in ReSharper 9.0.

**YouTrack:** [RSRP-415304](http://youtrack.jetbrains.com/issue/RSRP-415304)

**Details:** The implementation of `XmlDaemonBehavior` returns false for `RunInSolutionAnalysis` (as do the implementations of `ILanguageSpecificDaemonBehavior` for build scripts and DTD files). The reasoning behind this was that XML files are more likely to be very large, and cause performance issues when processed during SWEA. However, there is a now a mechanism in place to prevent processing overly large files, using `PerformanceThresholds` and `DoNotBuildPsiForHugeFiles`. 

## XML files are not passed to ICache implementations

**Problem:** Any XML or XML derived file is not passed to `ICache` implementations, meaning XML files cannot participate in background processing.

**Solution:** This will be addressed in ReSharper 9.0. However, there is a workaround, described in the Details section below.

**YouTrack:** [RSRP-416533](http://youtrack.jetbrains.com/issue/RSRP-416533)

**Details:** When passing files to implementations of `ICache`, ReSharper checks certain properties on the `IPsiSourceFileProperties` instance exposed by `IPsiSourceFile.Properties`. It ensures that `ShouldBuildPsi`, `ProvidesCodeModel` and `IsICacheParticipant` are all `true`. If not, the file is not processed.

XML files return `false` to `ProvidesCodeModel`. This flag will be changed for ReSharper 9.0.

It might seem more intuitive to simply check `IsICachePartipant` rather than checking three properties. The requirements for participating in `ICache` is the ability to parse a file and that the file contributes to the code model. The `IsICacheParticipant` property is actually an *opt out* mechanism, allowing files that satisfy the base requirements to opt out of `ICache`. It is required for C++ support.

The workaround is to implement `IPsiSourceFilePropertiesProvider`. The implementation should have an arbitrarily high value for `Order`, so that it executes after the default implementation which provides a standard implementation of `IPsiSourceFileProperties`. In its implementation of `GetPsiProperties`, it should check to see if the file is appropriate - that is, it's an XML file (`psiSourceFile.LanguageType.Is<Xml>()`) and if necessary, that the project is also appropriate (for example, if you're adding caching just for web projects, check the project is a web project before continuing). The method should then return an instance of `IPsiSourceFileProperties` that returns `true` for `ProvidesCodeModel`, and defers everything to the original `IPsiSourceFileProperties` passed in. If its not a file that you're interested in, simply return the original properties untouched.

## "Browser not found by alias 'C'" message when parsing web files in tests

**Problem:** When trying to run a test that needs to parse a web file, such as `.css`, `.html` or `.aspx`, the test fails with a message like "Browser not found by alias 'C'". The CSS inspection configuration data is missing, and this causes the CSS daemon to throw exceptions, failing the test.

**Solution:** Copy the `CSS` folder from the ReSharper install directory to the `bin\Debug` folder of your tests. It will be properly addressed in ReSharper 9.0

**YouTrack:** [RSRP-416928](http://youtrack.jetbrains.com/issue/RSRP-416928)

**Details:** The `CSS` folder in the product installation folder contains various configuration files defining supported browsers, and CSS properties. The CSS daemon requires these files in order to properly inspect CSS files. If they're missing, the daemon throws an exception, causing the tests to fail. Ensure the CSS folder is copied from the ReSharper install directory to the `bin\Debug` folder of your tests and the CSS daemon will start working properly again. A pre- or post-build step can accomplish this

## Project files are not handled by daemon stages

**Problem:** It is not possible to process `.csproj` or `.vbproj` project files in an instance of `IDaemonStageProcess`. They are not processed by the visible document daemon process (because they're not visible documents!) and are explicitly excluded from Solution Wide Error Analysis.

**Solution:** There is no workaround at present

**YouTrack:** [RSRP-418226](http://youtrack.jetbrains.com/issue/RSRP-418226)

## Cannot compile project that includes a license

*This issue concerns plugin developers who use a license file (licenses.licx) in their plugin project.* Since the license compiler (`LC.EXE`) takes as parameters the full path of each assembly reference, you may run into an exception if the sum total of all reference paths exceeds 32000 characters. Possible workarounds for this issue are:
    * Install the ReSharper SDK into a path that is as short as possible; or
    * Create your own `.Target` files, removing some of the entries that your plugin does not require.

