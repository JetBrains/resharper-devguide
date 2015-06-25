---
---

# dotPeek

This page contains _preliminary_ documentation on developing plugins for dotPeek. As dotPeek does not yet have an SDK, this guide should serve the temporary purpose of explaining how to write dotPeek plugins until an SDK is introduced.

All of the JetBrains .net tools share the Platform and PSI parts of the architecture, and share some parts of Features, so the existing documentation in this guide should cover writing plugins for dotPeek.

Major points of interest:

* dotPeek is obviously a standalone product, and therefore does not provide any integration with Visual Studio.
* The decompiler pipeline is not currently designed to be extensible.
* The Extension Manager functionality is not included in dotPeek. This means plugins need to be [manually installed](/Extensions/Packaging.md#manual-install).

The dotPeek binaries should be included directly in the plugin project. The [`JetBrains.DotPeek.PluginDevelopment`](https://www.nuget.org/packages/JetBrains.DotPeek.PluginDevelopment/) nuget package can automate this (this is an official nuget package).

To test the plugin, you can use the following command line:

    dotPeek.exe /Plugin foo.dll

