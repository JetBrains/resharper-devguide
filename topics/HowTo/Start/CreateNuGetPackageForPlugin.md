[//]: # (title: 2. Create a NuGet Package for the Plugin)

To create a distributable NuGet package for a plugin:
1. Download and install [NuGet](https://dist.nuget.org/index.html), e.g., to *c:\NuGet*.
2. Create a blank NuGet specification file (e.g., *myplugin.nuspec*) and place it to the plugin's project folder.
3. Edit the *.nuspec* file:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <package>   
     
        <metadata>        
            <id>MyCompany.MyPlugin</id>
            <version>0.0.1</version>
            <authors>Me</authors>
            <owners>Me</owners>
            <requireLicenseAcceptance>false</requireLicenseAcceptance>
            <description>My first ReSharper plugin</description>
                        
            <dependencies>
                <dependency id="Wave" version="[8.0]" />
            </dependencies>
                        
        </metadata>      
          
        <files>
            <file src="bin\Debug\MyPlugin.dll" target="dotFiles\" />
            <file src="bin\Debug\MyPlugin.pdb" target="dotFiles\" />
        </files>
                
    </package>
    ``` 
    What's important here:
    * The `id` parameter in metadata MUST consist of two parts (company and a plugin name) separated by `.` (dot): `MyCompany.MyPlugin` in our example.
    * `<dependency id="Wave" version="[8.0]" />` specifies the version of ReSharper your plugin targets. Thus, Wave 8.0 stands for ReSharper 2017.1, Wave 9.0 for ReSharper 2017.2, and so on. Note that we put the strict limitation on the required ReSharper version (by using the square brackets in `[8.0]`) – ReSharper SDK is not backward compatible!
4. Build the NuGet package by running:

    ```text
    nuget pack myplugin.nuspec
    ```
Once you have the package file, you can [set up the environment](SetUpEnvironment.md).