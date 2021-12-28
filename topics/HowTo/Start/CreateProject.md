[//]: # (title: 1. Create a Project)

To create a plugin project:
1. In Visual Studio, create a new project of the **Class Library** (Visual C#) type.
2. Reference ReSharper SDK. It is distributed via NuGet, so, you can reference the SDK via the package manager:
    * Open **Tools &#124; NuGet Package Manager &#124; Manage NuGet Packages for Solution…**.
    * In the opened window, find **JetBrains.ReSharper.SDK**.
    * In **Version**, select the package version. It must match the ReSharper version your plugin targets.
3. Click **Install**.
4. Build the project to get the *.dll* file.

Now, you should [create the NuGet package for the plugin](CreateNuGetPackageForPlugin.md).