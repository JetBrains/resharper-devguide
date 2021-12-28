[//]: # (title: Work with IProperty)

**What you should know beforehand:**
* [Lifetime](WorkWithLifetime.md)
* [Signals](WorkWithSignals.md)

`IProperty` is one more concept that is used quite often throughout the ReSharper API. Simply put, an `IProperty` object is able to track the changes, validate new values being assigned, and  notify about the changes. `IProperty` has much in common with [signals](WorkWithSignals.md).

## Basic Usage
**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [PropertyProvider.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/IProperty/PropertyProvider.cs)
* [PropertyTester.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/IProperty/PropertyTester.cs)

For example, the `SomeIntProperty` parameter of the `PropertyProvider` class is important for the work of `PropertyTester`. Once the parameter is updated, `PropertyTester` is notified about the new value:

```csharp
public class PropertyProvider
{
    public IProperty<int> SomeIntProperty;

    public PropertyProvider(Lifetime lifetime)
    {
        SomeIntProperty = new Property<int>(lifetime, "PropertyProvider.SomeIntProperty");
    }
}
 
 
public class PropertyTester
{
    private readonly PropertyProvider _propertyProvider;
 
    public PropertyTester(Lifetime lifetime, PropertyProvider propertyProvider)
    {
        _propertyProvider = propertyProvider;
        _propertyProvider.SomeIntProperty.BeforeChange.Advise(lifetime, args =>
        {
            if (args.New < 0)                
                args.Cancel = true;                
        });
 
        _propertyProvider.SomeIntProperty.Change.Advise(lifetime,
            val => MessageBox.ShowInfo($"New property value is {val}"));
    }
 
 
    public void ChangePropertyValue(int value)
    {
        _propertyProvider.SomeIntProperty.Value = value;
    }
}
```

>
>* As almost any other ReSharper API component, an `IProperty` instance has lifetime, which means ReSharper is in charge of the object disposal.
>* When the lifetime is terminated, all handlers of the property events are forcibly detached in order to prevent memory leaks.
>* `BeforeChange` is a signal that fires before the value changes. You can cancel the change by passing `Cancel = true` in signal arguments. The arguments also store old and new property value.
>* `Change` is a [signal](WorkWithSignals.md) that fires on value change. Use it to track the changes. In addition, there are multiple other methods that allow tracking changes, e.g., 
>    * `ForEachNewValue` - the handler will be called each time a new value is assigned
>    * `When` - the handler will be called when the value gets equal with a desired value
>* To change the value, use the `Value` property.
>
>
{type="note"}

## Global Options
**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [OptionsPageViewModel.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/Ui/OptionsPageViewModel.cs)

One more obvious example of how `IProperty` can be used is global options: once a user changes an option on the **Options** page, the plugin gets instantly notified about the change.

```charp
Text = new Property<string>(lifetime, "OptionsExampleViewModel.Text");
 
var checkMeOption =
    settingsStore.BindToContextLive(lifetime, ContextRange.ApplicationWide)
        .GetValueProperty(lifetime, (MySettingsKey key) => key.CheckMe);
 
checkMeOption.Change.Advise_HasNew(lifetime, v =>
{
    Text.Value = v.New ? "checked" : "not checked";
});
```

>
>* `checkMeOption` is a property (`IProperty`) that represents the `CheckMe` setting on the **Options** page.
>* Depending on the `CheckMe` setting state, the `Text` property is either "checked" or "unchecked".
>* The `Text` property is updated via `Advise_HasNew` and is used to show the `CheckMe` setting state on a WPF control (see the next section).
>
{type="note"}

## WPF Data Binding
**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [OptionsPageViewModel.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/Ui/OptionsPageViewModel.cs)
* [OptionsPageView.xaml](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/Ui/OptionsPageView.xaml)
* [OptionsPageView.xaml.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Options/Ui/OptionsPageView.xaml.cs)

`IProperty` implements `INotifyPropertyChanged` out of the box, so, it is able to participate in WPF data binding. For example, we can show the current value of an `IProperty` on a WPF control. Let's take a look at how to do this based on the example from the previous section. Say, we want to display the value of the `CheckMe` setting (located in **ReSharper &#124; Options...**). This is how the view model looks like:

```csharp
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

The code behind the view looks like follows:

```csharp
[View]
public partial class OptionsPageView : IView<OptionsPageViewModel>
{
    public new string Name => "Options";
 
    public OptionsPageView()
    {
        InitializeComponent();
    }
}
```

The XAML for the view:

```xml
<UserControl x:Class="SampleReSharperPlugin.OptionsPageView"
            ...
             xmlns:viewModels="clr-namespace:SampleReSharperPlugin"
             d:DataContext="{d:DesignInstance Type={x:Type viewModels:OptionsPageViewModel}, IsDesignTimeCreatable=False}">
 
    <Grid>
        <StackPanel>
            <TextBlock HorizontalAlignment="Left" TextWrapping="Wrap"
                       VerticalAlignment="Top" Margin="10,10,5,0">
                The option in <Bold>ReShaper > Options... > Tools > Sample R# Plugin > Sample bool option</Bold> is:
                <InlineUIContainer>
                    <TextBlock TextWrapping="Wrap" Text="{Binding Text.Value}" FontWeight="Bold" />
                </InlineUIContainer>
            </TextBlock>
        </StackPanel>
    </Grid>
</UserControl>
```

When creating a view and viewmodel, don't forget to specify data context:

```csharp
var propViewModel = new PropertyViewModel(lifetime);
var propView = new PropertyView { DataContext = propViewModel };
```

>
>* ReSharper provides its own set of classes used to simplify implementing WPF MVVM pattern: 
>    * `AAutomation` provides base implementation of the `INotifyPropertyChanged` interface and can be used for creating view models. 
>    * The view class should be marked with the `View` attribute and implement the `IView` interface.
>
>
{type="note"}

## Data Flow Between Properties
**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [PropertyFlow.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/IProperty/PropertyFlow.cs)

You can organize the data flow between two properties. The properties can be of the same type or of different types. In the latter case, you should write a type converter. For example, we have two string properties: the first one provides some string; the second one takes this string and applies to it Title Capitalization:

```csharp
public class PropertyFlow
{
    public IProperty<string> SourceProperty { get; set; }
    public IProperty<string> TargetProperty { get; set; }
 
    public PropertyFlow(Lifetime lifetime)
    {
        SourceProperty = new Property<string>(lifetime, "sourceProperty");
        TargetProperty = new Property<string>(lifetime, "targetProperty");
 
        SourceProperty.FlowChangesInto(lifetime, TargetProperty, s =>
        {
            if (string.IsNullOrEmpty(s))
                return "";
 
            var textInfo = new CultureInfo("en-US", false).TextInfo;
            s = textInfo.ToTitleCase(s); 
 
            return s;
        });
    }
}
```

## Color Themes
The classes responsible for working with color themes extensively use `IProperty`. 
For more details, refer to [Work with Color Themes](WorkWithColorThemes.md).