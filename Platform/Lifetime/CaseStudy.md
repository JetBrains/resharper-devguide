---
title: Case Study
---

> **NOTE** This topic refers to the implementation of the Extension Manager for ReSharper 8. This implementation has changed for ReSharper 9 and no longer matches this description. However, the described behaviour is still a very valid usage of the `Lifetime` classes.

`Lifetime` is a very powerful concept. Normal usage is very straight forward, either by adding a simple callback, or passing it to an API. Generally speaking, this is how it will be used day to day.

However, it can also be used to help manage more complex scenarios.

Let's take a deeper look at one such example. Consider ReSharper's Extension Manager. Extensions can be installed and uninstalled at any time. They can also be disabled and enabled, and there are also thread affinity concerns. `Lifetime` really helps keep the code manageable.

## Loading Extensions

Since extensions can be uninstalled at any time, it makes sense that they have a `Lifetime`. If an extension is uninstalled, its `Lifetime` is terminated. Any registered actions will be run and any nested `Lifetimes` will also be terminated.

## Settings Files

When an extension is first loaded, the `ExtensionSettingsLoader` class looks to see if it contains any settings files. If so, it calls into the Settings subsystem to load the files. The files are loaded with a `Lifetime` instance, and the Settings subsystem will register a callback that removes the settings file when that `Lifetime` terminates.

> **NOTE** Note that there is no code to explicitly remove settings files. It is automatically handled by nested `Lifetime`s and the settings subsystem adding cleanup code to the given `Lifetime`.

However, we also want to remove settings files when the extension is disabled. So, instead of calling the Settings subsystem with the extension's own `Lifetime`, `ExtensionSettingsLoader` creates a new `Lifetime` that is valid for the duration that an extension is enabled. Since this is a consecutive, non-overlapping duration, the settings loader uses an instance of `SequentialLifetimes` to manage this.

When the extension is enabled, `SequentialLifetimes.Next` is called to create a new `Lifetime`, and this can be passed to the Settings subsystem when loading the files. When the extension is disabled, `SequentialLifetimes.TerminateCurrent` causes this settings file `Lifetime` to be terminated, running the Settings subsystem's callback, which removes the files. 

Since `SequentialLifetimes` creates a nested `Lifetime` with the extension's own `Lifetime` as the parent, when the extension is uninstalled, this parent `Lifetime` terminates, which terminates the nested `Lifetime`, and the settings files are automatically removed.

## Enforcing Thread Affinity

To further complicate matters, there is a thread affinity issue here. When adding a settings file, it must be done on the main thread, since the list of settings layers are used as-is for binding to WPF (a design flaw, admittedly) and if they're not added on the main thread, WPF throws an exception. This requires the call into the `Settings` subsystem to be enqueued to run on the main thread.

However, we now run the risk of a race condition - if we enqueue an action to add the settings file on the main thread, it's possible that the extension is disabled or even uninstalled before the settings files can be loaded. 

Fortunately, we can use a `Lifetime` when enqueuing the add action to the UI thread. We can use the `Lifetime` created by the `SequentialLifetimes` for this. If the extension is disabled, or uninstalled, the `SequentialLifetimes` class will terminate the `Lifetime` and the queued action is not executed.

## Removing Settings Files

Removing the settings files is also complicated by the thread affinity issue.

`Lifetime` instances are not multi-threaded. That is, their callbacks are executed on the same thread that terminated the `Lifetime`. If there are nested `Lifetime` instances, then all callbacks of both the parent and the children are are executed on the same thread.

So, we can't add the settings files with a `Lifetime` that is a child of either the extension's own `Lifetime`, or the `SequentialLifetimes` class; if either of these terminate, then the callback is executed on the same thread that terminated it. This means the Settings subsystem will try to remove the settings on the same thread that either uninstalled or disabled the extension, and this is not necessarily the UI thread.

`Lifetime` doesn't provide any threading facilities, so we need to manage this ourselves. The answer is fairly straightforward though - create a new, unnested `Lifetime` (it's parent is `EternalLifetime.Instance`) and use it to load the settings file. This `Lifetime` is later explicitly terminated on the UI thread, by registering a callback with the `SequentialLifetimes` instance that calls `LifetimeDefinition.Terminate`.

The final thing to consider is the `Lifetime` used to enqueue the terminate action to the UI thread. Since this only happens when a `Lifetime` is terminated, we can't use any existing `Lifetime` instances (they've been terminated - the action wouldn't happen!), so in this case, we use `EternalLifetime.Instance` - we always want this to happen, because no-one else can terminate that unnested `Lifetime`. We also know that the action will be executed, and removed from the `EternalLifetime` instance, so there's no memory leak.

## Summary

This means we have three `Lifetime` objects on the go. 

* Firstly, the extension's own `Lifetime`. This is the main parent `Lifetime`. If it's terminated, all other `Lifetime` objects are also terminated. 
* Secondly, we have a `Lifetime` created by the `SequentialLifetimes` object every time the extension is enabled, and terminated when it's disabled. This is used to enqueue a call to the main thread, and also used to register a callback to explicitly terminate the third `Lifetime`.
* The third `Lifetime` is an un-nested `Lifetime` that is passed to the `Settings` subsystem. Terminating this is what causes the settings files to be unloaded. This `Lifetime` is explicitly terminated on the UI thread. The terminate action is enqeued to the UI thread with the `EternalLifetime` to ensure it actually happens.

We now have a system of adding and removing the settings files when the extension's `Enabled` property changes. We also add and remove the settings files on the main thread, and don't add the settings files if the extension has been disabled before the main thread was available to run the command. Finally, everything is removed when the extension is uninstalled.

Consider implementing these same requirements with `IDisposable`.

* Firstly, we would need to know when the extension was being uninstalled, which `IDisposable` wouldn't give us. (We would need to register with some event, presumably exposed by the extension itself)
* Secondly, we would have to manage state to know if the files were currently loaded or not in the Settings subsystem and update this when the extension's enabled state changed.
* Thirdly, we would have to manage enqueuing to the main thread, and work around the race condition of disabling the extension before adding the files.
* Finally, we would have to explicitly tell the Settings subsystem to unload the settings files when either the extension was removed, or made disabled.

`Lifetime` manages a lot of state for us in this situation. 

