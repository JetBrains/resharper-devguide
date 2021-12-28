[//]: # (title: Work with Color Themes)

**What you should know beforehand:**
* [Lifetime](WorkWithLifetime.md)
* [IProperty](WorkWithIProperty.md)

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [UiThemeViewModel.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/UiThemes/Ui/UiThemeViewModel.cs)
* [UiThemeView.xaml](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/UiThemes/Ui/UiThemeView.xaml)

Changing user interface colors depending on the selected theme can be automated by means of the `ColorThemeManager` component. For example, we want to change the background and foreground colors of some UI control:
```
public class UiThemeViewModel : AAutomation
{
    public IProperty<SolidColorBrush> BackgroundColor { get; set; }
    public IProperty<SolidColorBrush> TextColor { get; set; }
 
    public UiThemeViewModel(Lifetime lifetime, IColorThemeManager colorThemeManager)
    {
        BackgroundColor = new Property<SolidColorBrush>(lifetime, "UiThemeViewModel.BackgroundColor") {Value = new SolidColorBrush()};
        TextColor = new Property<SolidColorBrush>(lifetime, "UiThemeViewModel.TextColor") {Value = new SolidColorBrush()};
 
        var bgColor = colorThemeManager.CreateLiveColor(lifetime, ThemeColor.ToolWindowBackground);
        bgColor.ForEachValue(lifetime, (lt, color) => { BackgroundColor.Value.Color = color.WpfColor; });
 
        var txtColor = colorThemeManager.CreateLiveColor(lifetime, ThemeColor.ToolWindowForeground);
        txtColor.ForEachValue(lifetime, (lt, color) => { TextColor.Value.Color = color.WpfColor; });
    }
}
```

## Notes
* The `CreateLiveColor` method of `ColorThemeManager` returns `IProperty` that is bound to the value of a particular `ThemeColor`.
* The `ForEachValue` method of `IProperty` allows assigning the updated color each time it is changed. The main difference in comparison with the `Advise` method is that `ForEachValue` passes a per-value lifetime (`lt`). It is terminated each time a new value is assigned to the property and allows defining a pair of actions (after-value-comes - before-value-goes) for each new value. 