[//]: # (title: Running A Plugin)

<!-- Copyright 2000-2022 JetBrains s.r.o. and other contributors. Use of this source code is governed by the Apache 2.0 license that can be found in the LICENSE file. -->

A plugin generated from the template can be run through one of the [Run/debug configurations](https://www.jetbrains.com/help/rider/Run_Debug_Configuration.html) or from the command-line on Windows, macOS, and Linux.

## From Run Configurations

The included [run configurations](https://www.jetbrains.com/help/rider/Run_Debug_Configuration.html) are shortcuts for the aforementioned command-line invocations. Choose one of the following configurations in Rider and click <control>Run</control> or <control>Debug</control>:

* `Rider (Unix)` – Launches Rider on Unix-systems (macOS/Linux). Automatically compiles the frontend part using IntelliJ SDK.
* `Rider (Windows)` – Launches Rider on Windows. Automatically compiles the frontend part using IntelliJ SDK.
* `VisualStudio` – Launches Visual Studio with ReSharper.

![Plugin Run Configurations](run-configurations.png)

If your plugin has a frontend part, you will see a `Rider` configuration in IntelliJ IDEA as well that works cross-platform.

## From Command-Line

### Windows

On Windows, you can run the plugin in Rider by calling:

```bash
.\gradlew.bat
```

To launch the ReSharper plugin inside Visual Studio, call:

```powershell
.\runVisualStudio.ps1
```

### macOS & Linux

On macOS & Linux, you can run the plugin in Rider:

```bash
./gradlew :runIde
```
