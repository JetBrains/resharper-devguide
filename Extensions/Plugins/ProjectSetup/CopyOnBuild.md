---
---

# Copy Plugin on Build

When building a plugin, it must first be installed and registered with both ReSharper and Visual Studio, via the Extension Manager. However, as long as nothing changes that would alter that registration (such as adding new files, changing Visual Studio integration features, modifying Action registration), then the plugin files can be copied over for each build.

The SDK package provides functionality to automatically copy files across, but requires some set up (the SDK does not know which ReSharper installation you have installed into). For more details, see the general [Copy on Build](/Extensions/Deployment/LocalInstallation/CopyOnBuild.md) topic in the [Local Installation](/Extensions/Deployment/LocalInstallation.md) deployment guide for more details.
