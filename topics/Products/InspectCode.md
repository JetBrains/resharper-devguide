[//]: # (title: InspectCode Plugins)

This page contains documentation on developing plugins for [InspectCode](http://confluence.jetbrains.com/display/NETCOM/Introducing+InspectCode), part of [ReSharper Command Line Tools](http://confluence.jetbrains.com/display/NETCOM/Introducing+ReSharper+Command+Line+Tools).

InspectCode does not currently have a standalone SDK, however, it is compatible with ReSharper extensions, and will load them at runtime, if you pass the ID of the extension to the `/extension` command line argument (multiple arguments are accepted).

When implementing a plugin that you wish to use with InspectCode, care must be taken not to rely on any components or interfaces that rely on Visual Studio integration - InspectCode runs as a standalone command line application, outside of Visual Studio. Any components you create in the extension that requires a Visual Studio component should pass the `ProgramConfigurations.VS_ADDIN` argument to `ShellComponentAttribute` or `SolutionComponentAttribute`.

If you wish to create an extension that only targets InspectCode, the extension package should take a dependency on `"InspectCode"`, rather than `"ReSharper"`. In this way, InspectCode will load the extension, but ReSharper won't.

Currently, the only extension point for InspectCode that isn't part of ReSharper is to create a custom Issue logger.

* Create a standard ReSharper plugin, using the NuGet packages (target .NET 4.0)
* Add additional reference to `JetBrains.CommandLine.InspectCode.Unattended.dll` library from CLT distribution.
* Implement your own logic for `IInspectCodeConsumer`

  ```csharp
  public class IssueLogger : IInspectCodeConsumer
  {
    void IDisposable.Dispose() { }

    public void Consume(IIssue issue)
    {
      Console.WriteLine("{0}({1}) {2}", issue.File, issue.Range, issue.Message);
    }
  }
  ```

* Implement `IInspectCodeConsumerFactory` factory to return a new instance of your consumer

  ```csharp
  [SolutionComponent]
  public class IssueLoggerFactory : IInspectCodeConsumerFactory
  {
    public IInspectCodeConsumer CreateConsumer(IEnumerable<IProjectModelElement> inspectScope, FileSystemPath outputFile = null)
    {
      return new IssueLogger();
    }
  }
  ```

* Start InspectCode with following parameters

  ```powershell
  inspectcode ... /Plugin=path-to-custom-logger-dll
  ```