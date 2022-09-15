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

From the root directory, you can run your plugin inside ReSharper and Rider from the command-line:

<tabs group="build">

<tab title="Rider (Gradle)" group-key="gradle">

To launch Rider on Windows, you can start the `gradlew.bat` script:

```bash
.\gradlew.bat :runIde
```

To launch Rider on macOS or Linux, you can start the `gradlew` script:

```bash
./gradlew :runIde
```

</tab>

<tab title="ReSharper (PowerShell)" group-key="powershell">

To launch ReSharper in an experimental Visual Studio instance, you can start the `runVisualStudio.ps1` script:

```powershell
.\runVisualStudio.ps1
```

</tab>

</tabs>
