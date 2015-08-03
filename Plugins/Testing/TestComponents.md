---
---

# Test Components

> **WARNING** This topic assumes familiarity with the concept and usage of both the [Component Model](/Platform/ComponentModel.md) and [Zones](/Platform/Zones.md) topics.

When starting the [test environment](TestEnvironment.md), the [Component Model](/Platform/ComponentModel.md) will scan all of the assemblies in the output folder, including the ReSharper platform and product assemblies, as well as the plugin and its test assemblies. It will use the currently activated [Zones](Zones.md) to filter the components, and instantiate only those that are in active zones.

For the majority of components, this will be very similar to the normal ReSharper startup process. The test environment will likely be started with a zone definition that activates the standard testing zone definition `PsiFeaturesTestZone`, which in turn activates most of the zone definitions necessary for normal ReSharper functionality (e.g. C# and JS support, daemon, navigation, unit testing, etc.).

This process will also load any components that are defined in the plugin. Normal component zone usage applies, so a plugin's components will only be created if those components live in a namespace that has a [Zone Marker](/Platform/Zones/Usage.md#zone-markers) that in turn requires zones that are currently active. For example, the plugin could define components that live in a zone that requires the `IUnitTestingZone`. As long as the `IUnitTestingZone` is active, the plugin's components will also be active.

Since the test assembly is also scanned for components, it is possible to create components that are only loaded while testing. This can be very useful to provide services to tests, or to augment or override existing components in the main product. For example, it might be useful to replace an existing component that would hit the disk, or a database or web service in order to return a default, well known response. Existing components can be replaced, either by inheriting from that component, or by implementing the `IHideImplementation<TComponent>` interface. See the guide section on [overriding and replacing components](/Platform/ComponentModel/ContainersPartsCatalogues.md#overriding-and-replacing-components) for more details. If the component is one of many implementations of the same interface, and that type of component supports priority between components, it might be possible to override undesired behaviour by also implementing the interface and returning a higher priority.

In order for test components to be active, they need to live in a namespace (or below a namespace) that is marked with a [Zone Marker](/Platform/Zones/Usage.md#zone-markers) that requires an active zone definition. For example:

```csharp
namespace MyPlugin
{
  [ZoneMarker]
  public class ZoneMarker : IRequire<IMyTestsZone>
  {
  }

  [ShellComponent]
  public class MyTestComponent
  {
    // ...
  }
}
```
