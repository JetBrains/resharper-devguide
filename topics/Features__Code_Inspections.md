[//]: # (title: Code Inspections)

A central feature of ReSharper and Rider is to show [Code Inspections](https://www.jetbrains.com/help/rider/Code_Analysis__Code_Inspections.html) for potential quality issues, common practices and improvements, code redundancies, and other [inspection categories](https://www.jetbrains.com/help/rider/Code_Analysis__Code_Inspections.html#categories). Such code inspections are further classified by [severity levels](https://www.jetbrains.com/help/rider/Code_Analysis__Code_Inspections.html#severity), which determine how the code is underlined in the editor.

![Code Inspection](Features__Code_Inspections__Sample.png)

You can provide new code inspections by implementing:

- `IHighlighting` – an interface containing all the necessary data to show code inspections (and to enable quick-fixes)
- `ElementProblemAnalyzer` – a component that gets called by the SDK to analyze syntax trees and find highlightings

> You should declare a [zone marker]() that requires `PsiFeaturesImplZone`. For C# code inspections, you can also require `ILanguageCSharpZone`.
>
{type="note"}

## IHighlighting

The `IHighlighting` interface requires to implement 4 members, which return data specific to an individual occurrence of an inspection in code:

- `ToolTip` and `ErrorStripeToolTip` are the strings that are shown while hovering the inspection in the editor or the stripes on the right-hand side of the editor
- `IsValid()` determines whether the data connected with the inspection are still valid (usually PSI elements or text ranges)
- `CalculateRange()` returns the range in the document where the inspection should be shown 

## Static and Configurable Severities

Independent of an individual `IHighlighting` instance, the SDK requires general information about the inspection type, which is provided through the `StaticSeverityHighlightingAttribute` or `ConfigurableSeverityHighlightingAttribute`. As the names suggest, one is for highlightings with static severities and the other to let users change them:

- `ConfigurableSeverityId` is a unique [inspection identifier](https://www.jetbrains.com/help/rider/Code_Analysis__Code_Inspections.html#ids-of-code-inspections) used for severity configuration and suppression through comments
- `Languages` defines the languages (C#, ...) for which the inspection is intended
- `OverlapResolve` defines the policy when two inspections overlap
- `OverloadResolvePriority` defines the priority when two inspections with the same overlap policy and range collide

- [//]: # (- TODO: MK `ToolTipFormatString*`)

If you've applied the `ConfigurableSeverityHighlightingAttribute`, you also need to add the `RegisterConfigurableSeverity` with the following information:

- `Group` should reference a well-known identifier from `HighlightingGroupIds` (but you can also define your own group)
- `DefaultSeverity` is the pre-configured severity level of the inspection
- `Title*` represents the searchable title and is shown in the list of all inspections
- `Description*` is shown in the bottom field when the inspection is selected

## ElementProblemAnalyzer

The `ElementProblemAnalyzer<T>` component is responsible to examine all syntax trees in the solution and to create highlightings accordingly. The generic parameter `T` is used to constrain the elements on which the analyzer should run, e.g., with `ICSharpDeclaration`, it is only called for types and members. The `Run` is invoked by the SDK with the following parameters:

- `T element` is the start element from where the syntax tree is analyzed
- `ElementProblemAnalyzerData` provides additional context about the analysis
  - `Solution` and `File` for the analyzed element
  - `SettingsStore` for checking settings that influence the highlighting (except severity)
  - `ThrowIfInterrupted()` should be called in more complex analyzers
- `IHighlightingConsumer` is used to pass created highlightings out of the analyzer, usually `AddHighlighting`

>  When choosing a type for `T`, you should be as specific as possible to keep safe-casts and invocations to a minimum.
>
{type="tip"}

To allow the SDK working with the analyzer, it must be marked with the `ElementProblemAnalyzerAttribute` and include the following information:

- `ElementTypes` are the `ITreeNode` abstractions on which the analyzer is called; usually this is the same as `T`, but an analyzer can also implement the non-generic `IElementProblemAnalyzer`
- `HighlightingTypes` enumerates all highlighting types that are generated by the analyzer

## Sample Implementation

### Highlighting

<!-- snippet: sample-highlighting -->
```csharp
[RegisterConfigurableSeverity(
    SeverityId,
    CompoundItemName: null,
    Group: HighlightingGroupIds.CodeSmell,
    Title: Message,
    Description: Description,
    DefaultSeverity: Severity.WARNING)]
[ConfigurableSeverityHighlighting(
    SeverityId,
    CSharpLanguage.Name,
    OverlapResolve = OverlapResolveKind.ERROR,
    OverloadResolvePriority = 0,
    // Appears in solution-wide analysis
    ToolTipFormatString = Message)]
public class SampleDeclarationHighlighting : IHighlighting
{
    // Appears in suppression comments
    public const string SeverityId = "SampleDeclarationInspection";

    // Appears in settings
    public const string Message = $"ReSharper SDK: {nameof(SampleDeclarationHighlighting)}.{nameof(Message)}";
    public const string Description = $"ReSharper SDK: {nameof(SampleDeclarationHighlighting)}.{nameof(Description)}";

    public SampleDeclarationHighlighting(ICSharpDeclaration declaration)
    {
        Declaration = declaration;
    }

    // Used for IsValid and quick-fixes
    public ICSharpDeclaration Declaration { get; }

    public bool IsValid() => Declaration.IsValid();
    public DocumentRange CalculateRange() => Declaration.NameIdentifier.NotNull().GetHighlightingRange();

    // Appears in editor
    public string ToolTip => $"ReSharper SDK: {nameof(SampleDeclarationHighlighting)}.{nameof(Message)}";

    // Appears in scrollbar
    public string ErrorStripeToolTip => $"ReSharper SDK: {nameof(SampleDeclarationHighlighting)}.{nameof(ErrorStripeToolTip)}";
}
```
<!-- endSnippet -->

### ProblemAnalyzer

<!-- snippet: sample-problem-analyzer -->
```csharp
[ElementProblemAnalyzer(
    typeof(ICSharpDeclaration),
    // Allows to disable the problem analyzer if code inspection is disabled
    HighlightingTypes = new[] { typeof(SampleDeclarationHighlighting) })]
public class SampleDeclarationProblemAnalyzer : ElementProblemAnalyzer<ICSharpDeclaration>
{
    protected override void Run(
        ICSharpDeclaration element,
        ElementProblemAnalyzerData data,
        IHighlightingConsumer consumer)
    {
        if (element.NameIdentifier?.Name.All(char.IsUpper) ?? true)
            return;

        consumer.AddHighlighting(new SampleDeclarationHighlighting(element));
    }
}
```
<!-- endSnippet -->
