---
---

# Package the Extension

In order to test the extension against a custom instance (or even the main instance), it needs to be installed, and needs to be packaged up in a custom `.nupkg` file. This is detailed in the [Packaging](../../Packaging.md) guide.

> **NOTE** It might seem odd that a plugin needs to be packaged up and installed before it can even be run for the first time. However, the plugin must be registered with the ReSharper Platform in order to be loaded. If it's not registered, it can't be loaded. This pattern is also the same way that Visual Studio VSIX extensions work, however, the packaging is automated by custom tooling.
>
> For the current release of the ReSharper Platform, this process is manual. We hope to automate this for future releases.

