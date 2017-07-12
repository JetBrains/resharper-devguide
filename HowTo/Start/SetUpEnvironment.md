---
---

# 3. Set Up the Environment
The next step is to set up the environment so you can debug your plugin. We recommend that you have two different Visual Studio hives: one for plugin development and the second for debugging purposes.

To set up the environment for debugging the plugin:
1. Install ReSharper to a separate Visual Studio [hive](https://docs.microsoft.com/en-us/visualstudio/extensibility/the-experimental-instance). To do this, during the ReSharper installation, specify a hive name in **Options &#124; Experimental hive** (in the ReSharper installer). For this example, let’s use the `pluginDebug` hive name.
2. Run this Visual Studio hive. To do this, you will need to use the `/RootSuffix` argument. E.g., `devenv.exe /RootSuffix pluginDebug`
3. Before you can debug the plugin, you must initially install it to the Visual Studio hive you use for debugging. Do this via **ReSharper &#124; Extension Manager…**. Note that you will need to specify the folder with the plugin as a custom source. You can do this via **ReSharper &#124; Options… &#124; Extension Manager &#124; Add**.

    IMPORTANT! It's not necessary to reinstall the plugin each time you introduce the changes to your plugin; simple copying of the .dll will be enough in most cases (see the next steps). Nevertheless, if your changes affect any [integration points](/HowTo/IntegrateWithReSharper/IntegrateWithReSharper.md), such as actions, menus, or others, you MUST reinstall the plugin.
4. Close this Visual Studio instance. Now, you should configure your plugin project, so that the built *.dll* is copied to the "debug" Visual Studio hive each time you run the project.
5. Add to the plugin project's *.csproj* file the following lines:

    ```xml
    <PropertyGroup>
       <HostFullIdentifier>ReSharperPlatformVs14pluginDebug</HostFullIdentifier>
    </PropertyGroup>
    ```
    Here:
    * `Vs14` stands for the Visual Studio version (2015). For Visual Studio 2017 it would be `Vs15`.
    * `pluginDebug` stands for the Visual Studio hive name.

    This tells msbuild to copy the built *MyPlugin.dll* file to the *pluginDebug* Visual Studio hive (these lines are properties of a special CopyBuild step). 
6. Now, you should configure Visual Studio to run the plugin in a required hive after you run debugging.

    In Visual Studio you use for plugin development, open project's properties.
7. In **Debug**, choose **Start external program** and specify the path to Visual Studio, e.g., `C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\devenv.exe`.
8. In **Command line arguments**, add `/rootSuffix pluginDebug`. This will run the plugin in the *pluginDebug* hive after we run debugging.
9. Try running your plugin project. If everything is fine, the Visual Studio instance with the installed plugin must be run right after the build succeeds.
 
That’s it! Finally, you have:
* The plugin project.
* The NuGet configuration for building the plugin package.
* Two Visual Studio hives: one for development and one for debugging purposes.

You are ready to start developing your first ReSharper plugin! Now, you should decide what type of plugin you’re going to create. Depending on this, choose your way of [integrating with ReSharper](/HowTo/IntegrateWithReSharper/IntegrateWithReSharper.md).
