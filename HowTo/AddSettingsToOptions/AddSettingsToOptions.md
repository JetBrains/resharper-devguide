---
title: Add Settings to ReShaper Options
---

**What you should know beforehand:**
* [Lifetime](/HowTo/WorkWithLifetime.md)
* [IProperty](/HowTo/WorkWithIProperty.md)

**Examples:**
* [OptionsPage.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/OptionsPage.cs)
* [OptionsPageViewModel.xaml](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/Ui/OptionsPageViewModel.cs)

ReSharper settings are key-value based. Settings are like onions - they have layers and are applied top-down: 
1. **Predefined settings**: settings that come with ReSharper (can be modified by the next levels).
1. **Computer-level settings**: settings stored in files in user's `Localappdata`. All instances of ReSharper have access to this level.
1. **Solution-level**: settings stored in files next to the solution:
    * *.sln.dotSettings*: team-shared settings. This file is expected to be stored in VCS.
    * *.sln.dotSettings.user*: private user settings. Here you can store e.g., the last unit tests session, position of some dialog on the screen, and so on. So, these are settings that are specific to this particular solution, otherwise, they go to the computer level.
1. **Project-level**: settings stored in files next to the project. These settings may include, e.g. specific naming rules.
    * *.csproj.dotSettings*: team-shared settings. 
    * *.csproj.dotSettings.user*: private user settings.

## Adding a setting
To create a simple setting, you should derive your class from `SimpleOptionsPage` and mark it with the `OptionsPage` attribute:
```
[OptionsPage(Pid, "Sample R# Plugin", typeof(FeaturesEnvironmentOptionsThemedIcons.CodeInspections), ParentId = ToolsPage.PID)]
public class OptionsPage : SimpleOptionsPage
{
    private const string Pid = "MyPluginOptions";
 
    public OptionsPage([NotNull] Lifetime lifetime,
        [NotNull] OptionsSettingsSmartContext optionsSettingsSmartContext)
        : base(lifetime, optionsSettingsSmartContext)
    {
        IProperty<bool> checkMe = new Property<bool>(lifetime, "MyOptionsPage::SomeOption");
        checkMe.SetValue(
            optionsSettingsSmartContext.StoreOptionsTransactionContext.GetValue(
                (MySettingsKey key) => key.CheckMe));
 
        checkMe.Change.Advise(lifetime, a =>
        {
            if (!a.HasNew) return;
            optionsSettingsSmartContext.StoreOptionsTransactionContext.SetValue(
                (MySettingsKey key) => key.CheckMe, a.New);
        });
 
        AddBoolOption((MySettingsKey key) => key.CheckMe, "Sample bool option");
    }
}
 
  
[SettingsKey(typeof(EnvironmentSettings), "My settings")]
public class MySettingsKey
{
    [SettingsEntry(false, "Check Me")]
    public bool CheckMe;
}
```

### Notes

![resharper-options](setting-resharper-options.png)

* The `OptionsPage` attribute tells ReSharper that this is a ReSharper settings integration point.
* `"Sample R# Plugin"` passed to the attribute is the name of the options group.
* `AddBoolOption` adds the **Sample bool option** checkbox to the **Options** page.
* `MySettingsKey` is the class that describes the key that will be used to access your settings.
* The `CheckMe` field represents the settings storage entry that will be used to access the setting value.

## Reading a setting value 
To access the setting value, you can use the `ISettingStore` component:
```
public class OptionsPageViewModel: AAutomation    
{
    public IProperty<string> Text { get; set; }        
 
    public OptionsPageViewModel(Lifetime lifetime, ISettingsStore settingsStore)
    {
        Text = new Property<string>(lifetime, "OptionsExampleViewModel.Text");
 
        var checkMeOption =
            settingsStore.BindToContextLive(lifetime, ContextRange.ApplicationWide)
                .GetValueProperty(lifetime, (MySettingsKey key) => key.CheckMe);
 
        checkMeOption.Change.Advise_HasNew(lifetime, v =>
        {
            Text.Value = v.New ? "checked" : "not checked";
        });
    }
}
```
### Notes
* `BindToContextLive` sets the context for reading and writing values: in our example it's `ApplicationWide`. If `ContextRange.Smart` is specified, the context will depend on user's settings editing mode (see settings layers in the top of this section). 
