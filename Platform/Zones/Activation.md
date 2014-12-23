# Activation

Components are only included into the Component Model if the zones they belong to are "active". Zones are not activated by default, and must be activated by an activator class, decorated by the `[ZoneActivator]` attribute, and implementing `IActivate<TZone>` for each zone it activates.

> **NOTE** Components that only consume other zones do not need to handle activation. It is the responsibility of the required zones to handle their own activation. For example, a third party extension does not need an activator - as long as all of the zones it depends on are active, the extension is also active.

## Application startup

Activator classes are used very early on in application startup, and are responsible for providing the list of active zones that are used to filter the available components in the Component Model. Understanding how they are used during application startup is important to understand what zones are available once the application has started.

During application startup, the ReSharper Platform initially creates several default environment zones that are automatically activated. These zones are used to filter environment components, that is, components marked with the `[EnvironmentComponent]` attribute and used to populate the environment container used to bootstrap the application and create the shell container.

Activator classes are environment components (`ZoneActivatorAttribute` derives from `EnvironmentComponentAttribute`), and are therefore filtered by the active host environment zones (see the section on activating the activator below).

When creating the shell container, the list of active zones for the shell is calculated. This list begins with the default environment zones, and is added to by the list of zones being activated by each activator.

The list of zones is then expanded to include the zones it both inherits from and requires via `IRequire<TZone>`. It is also expanded to include zones that inherit *from* the zone being activated. This means `ILanguageCSharpZone` would be activated when the base `IPsiLanguageZone` is activated. Note that zones that simply require the zone being activated are not automatically activated.

Finally, the zones are filtered to remove any zones that violate licensing and that the user has explicitly disabled. This is now the final list of all zones that are active, and this is the list that is used to filter out shell and solution components.

## Usage

The `IActivate<TZone>` interface is defined as follows:

```cs
public interface IActivate<TZone>
  where TZone : IZone
{
  bool ActivatorEnabled();
}
```

Typically, a zone activator aligns with a product, and unconditionally enables all zones that the product requires. This may seem counter intuitive, however, if a product is installed, it should be activated. Also, the zone activator itself must be activated (see below), allowing products to be disabled, or only enabled based on environmental conditions. The typical activator will implement several `IActivate<TZone>` interfaces with a single instance of `ActivatorEnabled` that just returns true.

```cs
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
```

However, it is perfectly reasonable to perform operations in the `ActivatorEnabled` method. For example, the `InternalModeProductZoneActivator` class will activate the `IInternalVisibilityZone` only if the `/Internal` command line argument is supplied.

Similarly, multiple implementations of `IActivate<TZone>` can be created with explicit interface implementation:

```cs
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

> **NOTE** If a zone is not activated, components that require that zone will not be available in the application. If all of the components in an assembly are not available, that assembly isn't even loaded into memory, as the catalogues for the Component Model are based on caching and IL metadata, not Reflection.

## Automatically enabled zones

A zone definition can be declared as auto-enabled, if it makes sense that the zone is always available, regardless of configuration, installed products, etc. This is handled with the `ZoneFlags` parameter to the `[ZoneDefinition]` attribute.

```cs
[ZoneDefinition(ZoneFlags.AutoEnable)]
public interface IMyZone
{
}
```

This has the same effect as a simple implementation of `IActivate<TZone>.ActivatorEnabled()` that always returns true. However, it doesn't depend on any other zones being active, as activator classes do. This means it is always enabled.

Also, components decorated with a zone marker that does not include any zone definitions are always available, bypassing the need to be enabled (there are no zones, so how do they get enabled?).

## Inheritance and activation

When declaring a zone definition, dependencies can be declared either by implementing the `IRequire<TZone>` interface, or by inheriting directly from an existing zone definition. There is no difference between the two with regard to dependencies - when applying a zone to a class or namespace, that zone and all of its dependencies must be active before the component is allowed.

The difference comes with activation. When a zone inherits from another zone definition, it is automatically activated when the base zone is activated. For example, the `ReSharperZonesActivator` class implements `IActivate<IPsiLanguageZone>.ActivatorEnabled()` and returns true. This activates the `IPsiLanguageZone` and all zone definitions that inherit from it, including `IClrPsiLanguageZone` and therefore `ILanguageCSharpZone`. Any new code that is added that implements a zone inheriting from `IPsiLanguageZone`, or `IClrPsiLanguageZone` will also get automatically enabled.

Dependencies registered via `IRequire<TZone>` are **not** automatically enabled. These zones must be enabled via another activator, or via activation of their base zone definitions.

## Default activated zones

While most zones are not activated by default, the environment creates an initial set of active zones based on the current host environment. Code can use these zones to make components available or not depending on the hosting environment. The zones are:

* An environment zone, representing the currently running host environment, such as `IConsoleZone`, `ITeamCityZone`, or `IStandaloneUIZone`. When running in Visual Studio, the chosen zone derives from the `IVisualStudioZone` definition, depending on the version of the currently running instance. The hierarchy of zone definitions for Visual Studio allow specifying dependencies on just a single version, or anything since a specific version. E.g. `IJustVs12Zone` for Visual Studio 2013 (12.0) and `IJustVs11Zone` for Visual Studio 2012 (11.0), or `ISinceVs10Zone` for all versions since Visual Studio 2010 (10.0).
* A zone representing the currently running CLR version - `ISinceClr2Zone` or `ISinceClr4Zone`. Components can use this to make themselves only available for a particular version of the CLR.
* A zone representing the currently running CPU architecture. Either `IIntelCpuArchitectureZone` or `IAmd64CpuArchitectureZone`, which correspond to 32-bit and 64-bit respectively.

## Activating the zone activator!

The `[ZoneActivator]` attribute derives from `[EnvironmentComponent]`, meaning the activator is loaded into the bootstrap environment container during application startup. However, the environment container is itself filtered on active zones!

This means the zone activator is only available if it has been added to one of the default active environment zones (see above). In other words, there needs to be a zone marker either for the zone activator. For example, the `JetBrains.ReSharper.Product.Application.Product.ReSharperZonesActivator` class has the following zone marker:

```cs
namespace JetBrains.ReSharper.Product.Application
{
  [ZoneMarker]
  public class ZoneMarker : IRequire<IVisualStudioZone>
  {
  }
}
```

Note the namespace of the marker is above the namespace of the activator. This means that the activator belongs to the zone definitions declared in the marker's dependencies, namely `IVisualStudioZone`. So, the ReSharper product zone activator is only active when the `IVisualStudioZone` zone is itself active. As we've seen above, this zone is an environmental zone, and is created and activated before the environment container is composed.

