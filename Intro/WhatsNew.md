# What's New in ReSharper 9.0?

<!-- toc -->

## Zones and namespace refactorings

TODO

## Background Component Instantiation

Starting with ReSharper 9.0, the Component Model will no longer immediately create components, but will instead defer creation until the application becomes idle. This has a very positive impact on application startup performance.

More specifically, any classes marked with attributes such as `[ShellComponent]` and `[SolutionComponent]` are no longer created immediately, as the parts container is composed (e.g. application startup, or solution load), but are queued up to the current thread's `Dispatcher`, at `Background` priority. The components are still created on the main thread, as in previous versions, and all dependencies are still created first.

One implication of this change is that the application can now be up and running before all of the components in the container have been created. Previous versions would create the whole container before allowing the application to continue running. There may be cases where a component should be created in a more timely manner. For example, the Visual Studio integration needs to be created and hooked up before the application can continue. In this case, the component can require "instant" activation, in which case the component (and all of the components required to satisfy constructor dependencies) are created immediately rather than scheduled:

```cs
// Note the typo. This is likely to be fixed by RTM
[ShellComponent(Requirement = InstanciationRequirement.Instant)]
public class ImmediateComponent
{
}
```

See the [Component Model section](../Platform/ComponentModel.md) for more details.

Note that this only applies when the ReSharper platform is hosted in Visual Studio. Standalone products will create components immediately.

## Control Flow interface renaming

TODO

`ICSharpControlFlowGraf` -> `ICSharpControlFlowGraph`
`IControlFlowRib` -> `IControlFlowEdge`

## Bulk actions

TODO

## Actions integration

TODO

No more live-loading
