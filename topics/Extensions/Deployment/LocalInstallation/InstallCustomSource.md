[//]: # (title: Install From a Local Custom Source)

Once the extension has been initially packaged up, and a `.nupkg` file has been created, it must be installed locally into the ReSharper Platform. In order to do so:

1. Start Visual Studio, ensuring you are running in the appropriate experimental instance. E.g.

    ```
    devenv.exe /rootSuffix Plugins
    ```

    Of course, if the extension is for a standalone product, start the appropriate host, e.g. dotPeek (currently, the standalone products do not yet support extensions. This is planned for a future wave).

2. Add a custom source to the Extension Manager. Go to the Options dialog and select the Extension Manager page. Add a new source, pointing to a location on the local filesystem. Hit OK.
3. Display the Extension Manager, find and install the extension. The Extension Manager will aggregate extensions from the default gallery and the new custom source location (the local file system), displaying all applicable extensions. After being selected for install, the ReSharper Platform will invoke the unified installer to read/download the extension package, resolve any dependencies, and install and register the new pacakge(s). The host (e.g. Visual Studio) will need to be restarted to pick up the changes.