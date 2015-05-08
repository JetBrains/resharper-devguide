---
---

# Settings

Plugin writers can use a wide variety of mechanisms in order to store and retrieve program settings. However, ReSharper provides its own infrastructure for keeping options. The advantages of this approach are that

* Special arrangements exist which let the settings interact cleanly with options pages
* A uniform API makes settings easier to use in your components
* The settings infrastructure is conducive to settings migration, overlays, and other interactions

## Declaring settings

In ReSharper, settings are kept in ordinary classes that have been decorated with some metadata. Let us take a look at an example of a settings class:

```csharp
[SettingsKey(typeof(InternetSettings), "GitHub settings")]
public class GitHubSettings
{
  [SettingsEntry("", "GitHub username")]
  public string Username { get; set; }


  [SettingsEntry("", "GitHub password")]
  public string Password { get; set; }
}
```

As you can see, the above class is decorated with the `SettingsKey` attribute. This attribute is used to provide some information about what settings are kept, as well as their type. In the above example, the type of the settings is internet-related, and is indicated as such. You can explore ReSharper to find potential settings categories or, if you can't find a suitable type, simply use `System.Reflection.Missing` as the type.

Following the `SettingsKey` attribute, each property in the class is decorated with the `SettingsEntry` attribute. This attribute takes two parameters - the first being the default value for the property, the second the property's textual description.

> **NOTE** The default value specified here is perhaps mis-named. It's not actually a default value, but a value to use if the settings infrastructure is unavailable. If client code can't get hold of an instance of `ISettingsStore` to retrieve the value, it can use the default value in the attributes. To specify default values, your plugin should implement `IHaveDefaultSettingsStream`

## Reading and Writing Settings

Rather than reading the settings directly, the correct approach is to _bind_ settings to a particular context, and then operate on settings in this particular context. In order to bind settings, you first need an `ISettingsStore`, which can be injected into the required component via the constructor. Once you have acquired the `ISettingsStore`, you can bind it to a given data context (i.e. an `IDataContext`) using code similar to the following:

```csharp
var boundSettings = mySettingsStore.BindToContextTransient(ContextRange.Smart((lt, _) => context));
```

Subsequently, you can use the `GetKey()` method of `boundSettings` to access the settings you need. For example:

```csharp
var settings = boundSettings.GetKey<GitHubSettings>(SettingsOptimization.DoMeSlowly);
```

Now, when it comes to writing settings, there is a corresponding `SetKey()` method, but there is also a way to bind particular settings directly to UI (for example, when you are showing an options page). In order to do this, instead of injecting an `ISettingsStore`, you need to inject a class called `OptionsSettingsSmartContext`. This class has a `SetBinding` method that takes the following parameters:

* A `Lifetime` entity, which you can also inject into your options page.
* A reference to the UI control that needs to be bound. This is done differently for WPF and WinForms.
* A lambda in the form `(MySetting s) => s.SomeProperty` that tells the smart context which property to bind.

For example, to bind `GitHubSetting`’s `Username` property to a WPF `TextBox` called `usernameBox`, you could write the following:

```csharp
settings.SetBinding(lifetime, (GitHubSettings s) => s.Username, usernameBox, TextBox.TextProperty);
```

Things are a little different if you need to bind to a WinForms property: in thic case, you can use the `WinFormsProperty` helper class, particularly its `Create` method:

```csharp
settings.SetBinding(lifetime, (GitHubSettings s) => s.Password, WinFormsProperty.Create(lifetime, passwordBox, box => box.Text, true));
```

A small note on data contexts: as you have seen, the `BindToContextTransient` takes a data context as a parameter, which is fine if you’ve got an Action, but a bit more challenging if you need to read a value elsewhere. In this case, you can do the following:

* Inject a `DataContexts` (note the -s at the end) value into your component.
* Use `dataContexts.Empty` as the parameter.

## Working with Settings Layers

In addition to being able to manipulate settings individually, ReSharper also allows wholesale manipulation of settings using the concept of _layers_. A layer is simply a collection of settings that are stored in a particular file. The layering effect is such that layers above override the settings of layers below.

If you go into *ReSharper | Manage Options…* with an open solution, you will typically see three layers:

* A personal layer
* A team-shared layer
* A settings file stored on this computer

The sum total of settings is held in a registrar that can be affected using an API that is available to plugin writers. For example, if you have a separate settings file that you want to programmatically inject, you can use the `FileInjectedLayers` shell component to inject this file using code similar to the following:

```csharp
var host = context.GetData(DataConstants.InjectedLayersHost_IncludingHostItself);
var path = new FileSystemPath(filename);
fileInjectedLayers.InjectLayer(host.Value, path);
```

The above assumes that `context` is an available `IDataContext` and `fileInjectedLayers` is a shell component (acquired, e.g., by constructor injection) of type `FileInjectedLayers`.

