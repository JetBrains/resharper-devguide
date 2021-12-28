[//]: # (title: Advanced Configuration

While logging can be enabled on the command line), specifying a configuration file allows for much more flexibility in what gets logged. The file is an XML file called `LogConfiguration.xml`, and should be created in `%\LOCALAPPDATA%\JetBrains\{HostFullIdentifier}\v{WaveVersion}`.

 >  The `{HostFullIdentifier}` is the name of the install folder for the current host, such as `ReSharperPlatformVs12` for Visual Studio 2013. More details can be found in the section on [Host Identifiers](HostIdentifiers.md). Similarly, the `{WaveVersion}` is the version of the [ReSharper Platform Wave](PlatformVersioning.md), e.g. `v05` for ReSharper 10.0 or `v04` for ReSharper 9.2.
 >
 {type="note"}

When running the ReSharper Platform in an [experimental instance of Visual Studio](ExperimentalInstance.md), the logging subsystem will also look for a file called `LogConfiguration.Debug.xml`, allowing for a separate configuration for debugging extensions in experimental instances. This file is looked for first, and if it doesn't exist, the `LogConfiguration.xml` file is used instead.

 >  A separate log file can be used for tests, too. The test host use "Tests" as the host identifier, and "Test" as the sub-configuration suffix in the filename, which means tests will look for a file called `LogConfiguration.Test.xml` in the `%\LOCALAPPDATA%\JetBrains\Tests\v{WaveVersion}` folder. If this file doesn't exist, it will then look for `LogConfiguration.xml` in the same folder.
 >
 {type="tip"}

The best way to set up a log file is via the [Logging Options Page](Logging.md#logging-options-page) which is only visible when in [Internal Mode](InternalMode.md).

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

* `level` - this is the logging level, as text. See [here](Logging.md#logging-levels) for the list of logging levels.
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
        * `{temp}` - full path to the `%\TEMP%` folder.
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