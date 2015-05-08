---
---

# Logging

The ReSharper Platform provides a comprehensive set of logging features, such as logging levels and different log listeners. It can also be configured with different levels and listeners based on the log category, such as `JetBrains.DataFlow` for capturing log events from the DataFlow subsystem.

There are two ways to enable logging, via the command line and via a config file. The command line options are much simpler, and can only control logging level and enable logging to a file. The config file provides much more power and can configure multiple listeners, as well as different configuration options per category. ReSharper even provides an Options page to configure logging, once [Internal Mode](../Intro/InternalMode.md) is enabled.

## Command line

The command line options are very simple:

* `/LogFile` - enables logging to a file. The path to the file can be specified as an argument. If the file path isn't specified, it defaults to a file created in `%TEMP%\JetLogs` with a filename based on the current date, process name and process ID.
* `/LogLevel {level}` - sets the logging level. If this isn't specified, the default is `LoggingLevel.INFO` (see [below](#logging-levels) for more details on logging levels). Using this command line arg automatically enables logging. If `/LogFile` isn't specified, the default filename is used.
* `/Log` simply enables logging with the default values. This switch isn't available in ReSharper ([RSRP-430339](https://youtrack.jetbrains.com/issue/RSRP-430339)).

> **NOTE** As ReSharper is hosted in Visual Studio, it needs to prefix the command line args to prevent clashes with options for other Visual Studio extensions, or Visual Studio itself. This means the command line args are `/ReSharper.LogFile` and `/ReSharper.LogLevel`.
>
> Similarly, command line switches need to be registered with Visual Studio when the extension is installed. Due to an oversight, the `/ReSharper.Log` switch isn't registered, and so isn't available in ReSharper. The switch simply enables logging with the defaults, so specifying either of the other switches will also enable logging.

## Configuration file

Logging can also be enabled by creating a configuration file, which allows for much more powerful configuration. The file is an XML file called `LogConfiguration.xml`, and should be created in `%LOCALAPPDATA%\JetBrains\{HostFullIdentifier}\v01`. Alternatively, if the file is named `LogConfiguration.Debug.xml`, then it only applies when ReSharper is being run in a custom hive. The `LogConfiguration.xml` applies regardless of custom hive.

> **NOTE** The `{HostFullIdentifier}` is the name of the install folder for the current host, such as `ReSharperPlatformVs12` for Visual Studio 2013. More details can be found about the `HostFullIdentifier` in the section on [Extension Deployment](../Extensions/Deployment/LocalInstallation.md#updating-the-extension-locally).

The file format is fairly straightforward:

```xml
<configuration>

  <appender name="file" class="JetBrains.Util.Logging.FileLogEventListener">
    <arg>c:\logs\resharper-{date}.log</arg>
  </appender>

  <root level="VERBOSE">
    <appender-ref>file</appender-ref>
  </root>

  <logger name="JetBrains.Platform.Util" level="OFF" />

  <logger name="JetBrains.DataFlow" level="TRACE" additivity="false">
    <appender-ref>file</appender-ref>
  </logger>
</configuration>
```

The `appender` element defines an appender/listener, which is the class that will listen for events and output them.

* `name` - the `name` attribute is required so the appender can be referenced later in the file.
* `class` - the `class` attribute is the fully qualified type name of the appender class. The class must implement the `ILogEventListener` interface.
* `pattern` - the `pattern` attribute (not shown) defines a pattern that can be used to define the output pattern for messages. See [below](#appender-patterns) for more details.
* `arg` - the `arg` element is used to pass arguments to the constructor of the appender. Multiple child `arg` elements are allowed, and are passed as-is, as strings, to the appender. It is up to the appender to interpret the argument value. For example, the `FileLogEventListener` will substitute the current date in the value `C:\logs\resharer-{date}.log`.

The `root` element defines the logging level and appenders that apply for all categories, unless otherwise overridden. It is the default way of setting the logging details for the Platform.

* `level` - this is the logging level, as text. See [below](#logging-levels) for the list of logging levels.
* `appender-ref` - the `appender-ref` child elements specify the name of the appenders that are to be used to log events. The appenders need to be created in `appender` elements at the head of the file.

The `logger` element defines the logging level and appenders to use for a category. When creating a logger, the code can specify a category, usually a namespace. This allows for logging to be configured differently for different parts of the Platform. For example, the `JetBrains.Platform.Util` category can be set to `OFF` to disable all logging from components in the `JetBrains.Platform.Util` namespace, or the `JetBrains.DataFlow` namespace can be configured to output to a different file, and capture events at a different logging level.

* `name` - the `name` attribute is required to specify the category that is to be configured.
* `level` - the `level` attribute allows overriding the `root` logging level.
* `additivity` - the `additivity` attribute is a boolean value to control if previously declared appenders are also used for this category. The default value is `true`.
* `appender-ref` - the `appender-ref` child elements can be used to specify one or more appenders that will be used to output the log events. More than one element can be specified. All appenders must be specified in `appender` elements at the head of the file.

The ReSharper Platform will pick up changes to the file automatically, without requiring a restart.

### Built in appenders

The ReSharper Platform ships with a number of appenders:

* `JetBrains.Util.Logging.DebugOutputLogEventListener` outputs log events to an attached debugger, using the [`OutputDebugString`](http://msdn.microsoft.com/en-us/library/windows/desktop/aa363362.aspx) method.
    * It can accept a single string argument that defines a prefix to be output before each event.
* `JetBrains.Util.Logging.FileLogEventListener` outputs log events to a file.
    * It requires a single string argument specifying the file path. It will process the string and substitute special variables to make the log file unique.
        * `{pid}` - process ID
        * `{pname}` - process name
        * `{temp}` - full path to the `%TEMP%` folder.
        * `{date}` - the current date.
* `JetBrains.Util.Logging.MessageBoxListener` will display a message box when an event's message matches a given regular expression.
    * It requires two string arguments. The first is a regular expression that must match an event's message. The second is the text to be shown when the event matches.
* `LogEventListener` is a listener that will call a given `Action<LogEvent>` when an event is logged. It isn't intended to be used from the configuration file, but from code.
* `TextWriterLogEventListener` is also intended to be used by code, writing events to a given `TextWriter`.
* `ConsoleToolLogger`, `MsBuildToolLogger` and `TeamCityToolLogger` are all used by command line tools to log events when running in the command line, as MSBuild tasks, or in Team City. They are automatically configured by the console host.

### Appender patterns

The `pattern` attribute of the `appender` element can be used to format log messages. Control sequences beginning with the percent symbol `%` can be used to substitute parameters, e.g. `%L` is substituted with the log level.

* `%d` - date. Accepts option parameter in curly braces. E.g. `%d{hh:mm:ss}`
* `%m` - original message.
* `%M` - message, including exception with stack trace if present.
* `%e` - short exception.
* `%E` - long exception.
* `%n` - new line.
* `%l` - log level's first character, e.g. `V` for `VERBOSE`.
* `%L` - log level.
* `%c` - category (i.e. namespace). Accepts optional parameter in curly braces to limit length of namespace segments, e.g. `%c{2}` will output the category `A.B.C.D` as `C.D`.
* `%t` - thread ID.
* `%T` - thread name.
* `%p` - process ID.
* `%P` - process name.

Also, the width of the parameter can be specified between the percent symbol and the control character, e.g. `%{min}.{max}c` will ensure that the output value has at least `{min}` characters and at most `{max}`. If there are fewer characters than `{min}`, the text is right justified (padded with leading spaces). By specifying a negative number for `{min}`, the text is left justified (padded with trailing spaces).

### Logging levels

ReSharper supports 7 logging levels, from `OFF` to `TRACE`. These are defined in the `LoggingLevel` enum:

```csharp
public enum LoggingLevel
{
  OFF = 0,
  FATAL = 1,
  ERROR = 2,
  WARN = 3,
  INFO = 4,
  VERBOSE = 5,
  TRACE = 6
}
```

When logging is enabled, the default/recommended level is `INFO`, which is intended for normal event logging. The `VERBOSE` level is intended to capture debug information, and `TRACE` is for capturing events that are even more verbose than `VERBOSE` - such as call stacks, etc.

By specifying a logging level, you will get all log events at that level and below. So, `INFO` will also get the log events for `WARN`, `ERROR` and `FATAL`.

### Logging options page

The simplest way to create a file is to use the Logging options page when running in [Internal Mode](../Intro/InternalMode.md). This page shows the current configuration of logging, based on the command line args, the currently created and active log files, and what appenders/listeners and loggers are active.

<!-- TODO: Add screenshot -->

From this page, it is possible to create a log file, by clicking the "Create File" link next to the Common or Hive file details. Clicking "Create File" for Common will create the `LogConfiguration.xml` file, while clicking "Create File" for Hive will create the `LogConfiguration.Debug.xml` file. The generated file contain a default setting for `VERBOSE` logging to the `DebugOutputLogEventListener`, which outputs to an attached debugger. It also provides commented-out example syntax to configure file appenders and category-based logging.

The page also shows the current active configuration, including currently active logging command line args, listed appenders and which appender is applied to what category, at what level. It's a very handy at-a-glance view of how the logging is configured. It also includes a "Log Log" window, which is the logger used to log internal events from the logging system (to prevent recursion or loss of information when errors occur while logging).

### System debug and trace

The ReSharper Platform will add a `TraceListener` to route .net generated events through ReSharper's logging system.
