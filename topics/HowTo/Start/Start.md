[//]: # (title: Start)

ReSharper can be extended in two ways:
* **Declarative extension**: extensions that are distributed via settings files, e.g. live templates or structural search and replace patterns. They do not require any programming and are not the subject of this **How To**. 
* **Compiled extensions or plugins**: compiled assemblies that are loaded into the ReSharper process and have almost full access to the ReSharper API. Plugins are able to do the same things as ReSharper does - quick-fixes, context actions, refactoring, code injections, and others.

## Quick facts about ReSharper plugins
* A ReSharper plugin is a compiled C# assembly.  So, it's simply a *.dll* file.
* The plugin must be distributed as a NuGet package - a *.nupkg* file.
* Plugins are installed into Visual Studio using **ReSharper &#124; Extension Manager...**
* If you want to make your plugin available to all ReSharper users, you should upload it to the [ReSharper repository](https://resharper-plugins.jetbrains.com).
* Plugins are dependent on a ReSharper version. Thus, if a new *major* ReSharper version is released and you want your plugin to support it, you must rebuild the plugin for this specific version. Also, you should take into account that each ReSharper release may introduce some breaking changes. So, there's a possibility your plugin won't work with a new release out of the box.

To start developing a plugin, you should:
1. [Create a plugin project](CreateProject.md).
2. [Create a NuGet package for the plugin](CreateNuGetPackageForPlugin.md).
3. [Set up the environment for running and debugging the plugin](SetUpEnvironment.md).