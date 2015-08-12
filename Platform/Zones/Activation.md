---
---

# Activation

Components are only included into the Component Model if the zones they belong to are "active". Zones are not activated by default, and must be activated by an activator class, which is a component decorated by the `[ZoneActivator]` attribute, and implementing `IActivate<TZone>` for each zone it activates.

If a zone is not activated, any components that require that zone will not be available in the application. It's worth noting that if none of the components in an assembly are available, then the assembly itself isn't even loaded into memory, as the catalogues that the Component Model use are based on caching and direct IL metadata reading, rather than Reflection.

> **NOTE** Components that only consume other zones do not need to handle activation. As long as the zones it requires are activated, the component will be created by the Component Model. It is the responsibility of the required zones to handle their own activation.
>
> For example, a third party extension typically does not need an activator - as long as all of the zones it depends on are active, the extension is also active. However, if the extension defines a zone (perhaps to declare a [Feature or Product](FeaturesProducts.md)), then that zone must be activated, or none of the components that require this zone will be activated.

## Implementation

An activator class implements the `IActivate<TZone>` interface, which is defined as:

```csharp
public interface IActivate<TZone>
  where TZone : IZone
{
  bool ActivatorEnabled();
}
```

The class can implement the interface multiple times, once for each zone it activates. During application startup, the ReSharper Platform will query the activator for each implementation of `IActivate<TZone>` and call the `ActivatorEnabled` method. If the class returns `true`, the zone is marked as active.

When activating a zone, the ReSharper Platform activates that zone, and all zones that inherit from it. For example, if `IActivate<IPsiLanguageZone>.ActivatorEnabled()` returns true, the `IPsiLanguageZone` is now activated, as are all zone definitions that inherit from `IPsiLanguageZone`, such as `IClrPsiLanguageZone`, `ILanguageCSharpZone` and `ILanguageJavaScriptZone`.

Typically, a zone activator aligns with a [Product or Feature](FeaturesProducts.md), and unconditionally enables the zone definition that declares the Product or Feature via the `[ZoneDefinitionProduct]` attribute. It should also activate the zones that the Product or Feature zone requires. It is not an error to activate the same zone from multiple activators. E.g. if both ReSharper and dotPeek need the `ExternalSourcesZone`, they should both activate it - if they don't, and they end up being the only product installed into the ReSharper Platform, the required zone won't be active and the Product or Feature's requiring zones won't be loaded.

```csharp
[ZoneActivator]
class DotPeekZoneActivator :
  IActivate<DotPeekProductZone>,
  IActivate<ExternalSourcesZone>,
  IActivate<NavigationZone>,
  IActivate<DaemonZone>,
  IActivate<ILanguageCSharpZone>
{
  public bool ActivatorEnabled()
  {
    return true;
  }
}
```

There is usually little or no logic in the implementation of `ActivatorEnabled`. Generally speaking, if a product is installed, it should be activated. Activation is separate to licensing, trial versions and user disabling of a product or feature - this is handled separately by the ReSharper Platform. Unlicensed, expired, or disabled zones are removed from the list of activated zones before components are filtered into the container.

However, it is perfectly reasonable to implement some logic in this method, for example, the `InternalModeProductZoneActivator` class will activate the `IInternalVisibilityZone` (for [Internal Mode](/Extensions/InternalMode.md)) only if the `/Internal` command line argument is supplied.

### Explicit implementation

The activator can implement the `IActivate<TZone>` interfaces explicitly, to allow for different implementations for each zone:

```csharp
[ZoneActivator]
public class MyZone : IActivate<IZone1>, IActivate<IZone2>
{
  bool IActivate<IZone1>.ActivatorEnabled()
  {
    return ShouldActivateZone1();
  }

  bool IActivate<IZone2>.ActivatorEnabled()
  {
    return ShouldActivateZone2();
  }

  // ...
}
```

### Activator class as component

Similarly, zone activator classes are components (`ZoneActivatorAttribute` derives from `EnvironmentComponentAttribute`, so they are components that get created very early in the application lifecycle), which means they can accept dependencies in the constructor. For example, a Feature that is intended for use only in [Internal Mode](/Extensions/InternalMode.md) can do something like:

```csharp
[ZoneActivator]
public class MyInternalModeFeatureZonesActivator : IActivate<IMyInternalModeFeatureZone>
{
  private readonly InternalModeProductZoneActivator internalModeActivator;

  public MyInternalModeFeatureZonesActivator(InternalModeProductZoneActivator internalModeActivator)
  {
    this.internalModeActivator = internalModeActivator;
  }

  public bool ActivatorEnabled()
  {
    // Only enable this zone if the internal zone is going to be enabled
    return ((IActivate<IInternalVisibilityZone>)internalModeActivator).ActivatorEnabled();
  }
}
```

This is perhaps a contrived example - an internal only Feature would be better requiring `IInternalVisibilityZone` in its zone, rather than handling it in this manner. However, it is an example that serves as a reminder that a zone activator is a component, and follows normal Component Model rules.

### Component Model considerations

Since the activator class is a standard component, it naturally follows normal Component Model creation rules. This means that an activator class must itself belong to a zone that is already active!

In other words, the zone activator is only available if it has a zone marker. The zone marker can be an [empty zone marker](Usage.md#empty-zone-marker), in which case the activator is always active, and always created. Or it can require one or more of the default active zones (see below), in which case the activator component will only be active if all of the required zones are active.

For example, the `ReSharperZonesActivator` class, which lives in the `JetBrains.ReSharper.Product.Application.Product` namespace, has the following zone marker:

```csharp
namespace JetBrains.ReSharper.Product.Application
{
  // This zone is only active when IVisualStudioZone is active.
  // So any activator class in this zone is also only created when
  // running in Visual Studio.
  [ZoneMarker]
  public class ZoneMarker : IRequire<IVisualStudioZone>
  {
  }
}
```

Note the namespace of the marker is above the namespace of the activator. This means that the activator class belongs to the zone definitions declared in the marker's dependencies, namely `IVisualStudioZone`. So, the ReSharper product zone activator is only active when the `IVisualStudioZone` zone is itself active. This zone is an environmental zone, and is created and activated before the environment container is composed.

> **WARNING** If the activator class itself doesn't have belong to an active zone, it will not be created, and therefore not activate its zones. An activator class must have a [zone marker](Usage.md#zone-markers). This can be an [empty zone marker](Usage.md#empty-zone-marker), in which case it will be always created, or a zone marker that requires one of the default active zones (described below), in which case it will only be active if those required zones are active.

### Disabling activators

A zone activator can take a dependency on a zone, using `IRequire<TZone>` in the same way zone markers do. This allows for a zone activator to be disabled if any of these required zones are disabled, for whatever reason - unlicensed, expired or explicitly disabled by the user.

It is not immediately obvious why a zone activator would wish to be disabled.

This happens naturally if an activator's zone marker's dependencies aren't satisfied. However, zone markers for zone activators are limited in that they can only require the bootstrap zones that are used when creating the environment component container (see the section on [default active zones](Activation.md#default-active-zones)). It also happens if any of the zones being activated are no longer licensed.

The main reason for wanting to disable a zone activator is so that supporting zones are not activated unnecessarily. A common pattern is for a zone activator to require a dependency on the main zone it is trying to activate. If that zone has been disabled, for whatever reason, the activator is also disabled, and supporting zones are not activated.

For example, the ReSharper C++ activator requires the C++ product zone, but also activates code editing, daemon and navigation zones. If the C++ product zone feature is disabled, the C++ activator would still activate code editing, daemon and navigation. If no other Features or Products used those zones, they are unnecessarily activated (and probably unlicensed). By requiring the C++ product zone, the ReSharper Platform will disabled the C++ activator if the C++ product zone is disabled. And now, the code editing, daemon and navigation zones are only activated if another activator needs them.

```csharp
[ZoneActivator]
class CppProductZonesActivator :
  IActivate<ILanguageCppZone>,
  IActivate<ICodeEditingZone>,
  IActivate<DaemonZone>,
  IActivate<NavigationZone>,
  IActivate<ICppProductZone>,
  IRequire<ICppProductZone>
{
  public bool ActivatorEnabled()
  {
    return true;
  }
}
```

## Default active zones

The ReSharper Platform provides several default zones that are automatically activated based on the current host's environment. These zones are used to bootstrap the zone activators. An activator's zone marker can require one or more of these zones to allow for conditionally including and excluding zone activators based on the host environment.

For example, if a zone needs to be run inside a particular version of Visual Studio, the zone activator's zone marker can require the Visual Studio zone. If the current host is an appropriate version of Visual Studio, the Visual Studio zone is activated, the zone marker's dependencies are satisfied, and the zone activator is created and finally activates the target zone.

The default zones derive from `IHostSpecificZone` and are:

* An environment zone, representing the current running host environment. These zones derive from `IEnvironmentZone`
    * `IConsoleZone` when the host is a command line utility. Derived interfaces represent the different supported console applications, such as InspectCode and DuplicatesFinder.
    * `IStandaloneUIZone` for when the host is a standalone GUI application, such as dotPeek. Again, derived interfaces represent standalone applications.
    * `ITeamCityZone` for when the host is running inside TeamCity.
    * `IVisualStudioZone` for when the host is Visual Studio.
* CLR version zone - `ISinceClr2Zone` or `ISinceClr4Zone`. Components that are built for .net 4 should require `ISinceClr4Zone`. Note that `ISinceClr4Zone` has a dependency on `ISinceClr2Zone`, so all CLR 2 components are also available for .net 4.
* CPU architecture zone, representing 32 and 64 bit - `IIntelCpuArchitectureZone` and `IAmd64CpuArchitectureZone`.
* `ITestsZone` to indicate that the code is running under test.

The Visual Studio zones make extensive use of zone inheritance to model different versions. Each version has two interfaces, a "since" zone and a "just" zone, e.g. `ISinceVs10Zone` and `IJustVS10Zone`.

The "since" zones inherit from the previous version's "since" zone, so `ISinceVs12Zone` derives from `ISinceVs11Zone`, which derives from `ISinceVs10Zone`, etc. When running in VS12 (Visual Studio 2013), the `ISinceVs12Zone` is active, and so is `ISinceVs11Zone` and `ISinceVs10Zone`. If a component uses a Visual Studio feature that works across versions, it should require the zone at which the feature was introduced. For example, if the feature was introduced in Visual Studio 2012 (VS11), the component should require `ISinceVs11Zone`. When running in Visual Studio 2012 (VS11) or 2013 (VS12), the `ISinceVs11Zone` is active, and the component runs. If running in Visual Studio 2010 (VS10), the `ISinceVs11Zone` isn't active, and the component isn't loaded.

The "just" zones inherit from the **current** version's "since" zone, so `IJustVs12Zone` derives from `ISinceVs12Zone`. If code is written that targets just a single version of Visual Studio, it should require the "just" zone. For example, if code targets just Visual Studio 2010 (VS10), it should require `IJustVs10Zone`. When run in Visual Studio 2012 (VS11), the `IJustVs10Zone` isn't active, and the component isn't loaded.

## Inheritance and activation

When declaring a zone definition using the `[ZoneDefinition]` attribute, dependencies can be declared either by implementing the `IRequire<TZone>` interface, or by inheriting directly from an existing zone definition. There is no difference between the two with regard to dependencies - when applying a zone to a class or namespace, that zone and all of its dependencies must be active before the component is allowed.

The difference comes with activation. When a zone inherits from another zone definition, it is automatically activated when the base zone is activated. For example, the `ReSharperZonesActivator` class implements `IActivate<IPsiLanguageZone>.ActivatorEnabled()` and returns true. This activates the `IPsiLanguageZone` and all zone definitions that inherit from it, including `IClrPsiLanguageZone` and therefore `ILanguageCSharpZone`. Any new code that is added that implements a zone inheriting from `IPsiLanguageZone`, or `IClrPsiLanguageZone` will also get automatically enabled.

Dependencies registered via `IRequire<TZone>` are **not** automatically enabled. These zones must be enabled via another activator, or via activation of their base zone definitions.

The Visual Studio zones are very hierarchical (see above). The "just" zones are the zones that are activated. Because the "just" zone derives from a chain of "since" zones, when a "just" zone is activated, the derived "since" zones are also enabled. So when `IJustVs12Zone` is activated, it's inherited `ISinceVs12Zone` is also activated, as are `ISinceVs11Zone`, `ISinceVs10Zone`, `IVisualStudioZone`, etc.

## Auto-enabled zones

A zone definition can be declared as auto-enabled, if it makes sense that the zone is always available, regardless of configuration, installed products, etc. This is handled with the `ZoneFlags` parameter to the `[ZoneDefinition]` attribute.

```csharp
[ZoneDefinition(ZoneFlags.AutoEnable)]
public interface IMyZone
{
}
```

This has the same effect as a simple implementation of `IActivate<TZone>.ActivatorEnabled()` that always returns true. However, it doesn't allow for finer control, such as activating supporting zones - it will only activate `IMyZone`. If `IMyZone` requires other zones to be active, an activator class should be used.

> **NOTE** A zone definition is not always necessary. It should only be used when the zone needs to be referred to externally, either by a consuming component (the zone represents a reusable API), or by the [Products and Features](FeaturesProducts.md) page (the zone represents a user-facing feature or product that can be disabled). If a zone definition is not required, zone markers will ensure that the components are created correctly, when all required zones are active, or with an [empty zone marker](Usage.md#empty-zone-marker) that is always active.

&nbsp;

> **WARNING** Currently (as of Wave01), auto-enabled zone definitions cannot be used for creating [Products or Features](FeaturesProducts.md). The options page to control enabling/disabling Products and Features does not take auto-enabled zones into account, and requires a zone to be activated by an explicit activator class. If it doesn't find an activator for a Product or Feature, the checkbox is greyed out and the Product or Feature cannot be disabled.
>
> (The code is checking the hierarchical nature of the Product and Feature zone definitions so that when a Feature is disabled, any dependent Features are also disabled. It is using the active zones from the activators to get this information. Because none of the activators activate the auto-enabled zone, it appears to be a deactivated zone, and so the dialog marks the checkbox as disabled, even though the Feature is still enabled.)
>
> In order to successfully appear on the Products and Features page, a zone must be activated by a zone activator. Make sure to include a valid zone marker for the activator class!
>
> This will hopefully be fixed in a future wave.
