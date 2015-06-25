---
---

# Install to an Experimental Instance

This first step is optional, but strongly recommended for working with extensions for ReSharper. This allows running the plugin in a separate instance to the one used to build it. Doing this prevents the main instance from locking the plugin in memory, which would require restarting both the test Visual Studio and the editing Visual Studio. This is not strictly required for declarative extensions as ReSharper will not keep a lock on .dotSettings or .xml files, however, it may be useful for testing purposes. Similarly, this is not required for plugins for standalone products, such as dotPeek.

Visual Studio's ["Experimental Instance"](http://msdn.microsoft.com/en-us/library/bb166560.aspx) feature is intended for developing and debugging Visual Studio extensions, and maintains a separate copy of the configuration needed to run Visual Studio. Each experimental instance can have an entirely different configuration, from theme and window layout to the extensions that are loaded.

> **NOTE** "Experimental instances" were previously known as "custom hives"

By default, the ReSharper Platform and all Product extensions (dotTrace, dotPeek, etc) are installed as a per-user extension for the main instance of Visual Studio. It is not visible in other (experimental) instances. However, it can be installed into an experimental instance to allow for a separate version of ReSharper (complete with separate versions of extensions) than what is installed in the main instance.

To install to an experimental instance, run the ReSharper unified installer, select the Options button, and enter the name of the instance. The experimental instance does not need to exist before starting the install.

Once the install is complete, a separate instance of ReSharper has been installed to the system. The main instance of Visual Studio will continue to use the original install of ReSharper. Running Visual Studio in the new experimental instance will run with this newly installed copy of ReSharper, and only use the extensions installed into that copy.

To run Visual Studio in an experimental instance called "Plugins", run:

```
devenv.exe /rootSuffix Plugins
```

Of course, the experimental instance can be called anything, and you can install more than instance if you wish.

