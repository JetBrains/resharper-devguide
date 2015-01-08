# Usage

Each component must be marked as belonging to one or more zones before it will be added to the Component Model. Zones are declared with zone definition types, and zone markers are used to associate components with zone definitions.

## Zone definitions

A zone is declared by a type implementing the empty `IZone` marker interface, and decorated with the `[ZoneDefinition]` attribute. The type can be a class or an interface, with no significance attached to the choice, although it has implications on inheritance. The simplest definition declares a zone with no dependencies:

```cs
[ZoneDefinition]
public interface ISimpleZone : IZone
{
}
```

Zone definitions are hierarchical, and can specify dependencies on other zones that are required in order for that zone to work. For example, the `IPsiLanguageZone` requires the `PsiFeaturesImplZone` which in turn requires the `ITextControlsZone`, `IProjectModelZone` and `IDocumentModelZone`. Any component that requires the `IPsiLanguageZone` implicitly also requires all of its dependencies. All zone definitions in this dependency hierarchy must be active for the component to be added to the Component Model.

The above example declares a zone with no dependencies - all components that belong to this zone can be instantiated without requiring any other zone to be active.

Dependencies can be defined either by inheritance or by implementing the `IRequire<TZone>` interface. The two methods are essentially the same, except inheritance also implies that depending zones are automatically activated. This is discussed in more detail in the [Activation](Activation.md) section. Generally speaking, unless there is an "is-a" relationship between zones (e.g. `ILanguageCSharpZone` is-a `IPsiLanguageZone`) dependencies should be implemented with the `IRequire<TZone>` marker interface. The two approaches can be mixed, as appropriate.

If a zone definition is expected to be inherited, it makes sense to implement it as an interface. In this way, the inheriting zone definition has the ability to inherit from multiple zones. If the base zone definition is a class, the inheriting zone definition can only inherit a single zone.

```cs
[ZoneDefinition]
public interface IMyZone : IZone, IRequire<NavigationZone>
{
}
```

This example states that `IMyZone` has a dependency on `NavigationZone` (and transitively, on anything that `NavigationZone` depends on, such as the PSI, results lists and options dialogs).

A zone definition has to be activated before it can be used. Typically, the owning product will be responsible for activating all of the zones in the product. For example, the `ReSharperZonesActivator` activates all zones used by ReSharper. When creating an extension that just consumes zones, activation is not needed, as the consumed zones will already be activated (or not, in which case the extension should not, and will not be loaded). However, if the extension creates zone definitions, it must activate them. This can be done either with explicitly with an activator class, or via the `ZoneFlags.AutoEnable` parameter to the `ZoneDefinitionAttribute` constructor. More details, including the differences between the approaches, can be found in the [Activation](Activation.md) guide.

## Zone markers

Zone markers are used to declare that a component belongs to one or more zones. This means that the component can only be instantiated if all of those zones (and transitively, all dependent zones) are active in the product.

A zone marker is typically implemented by creating a type called `ZoneMarker`, or with a name ending in `_ZoneMarker`, and decorating it with the `[ZoneMarker]` attribute. All dependencies are listed either by implementing the `IRequire<TZone>` interface, or by passing the zone definitions to the `ZoneMarkerAttribute` constructor. There is no difference in the approach.

```cs
namespace Foo.Bar
{
  [ZoneMarker]
  public class ZoneMarker : IRequire<IMyZone>
  {
  }

  // or...
  // [ZoneMarker(typeof(IMyZone))]
  // public class ZoneMarker
  // {
  // }
}
```

This example declares that the `Foo.Bar` namespace belongs to the `IMyZone` zone. All components in the `Foo.Bar` namespace and below (e.g. both `Foo.Bar.ThisComponent` and `Foo.Bar.Baz.Quux.ThatComponent`) require the `IMyZone` zone, and all of its dependencies to be active, or they do not get added to the Component Model.

Multiple dependencies can be applied to the zone marker, meaning the components in the zone require multiple zones for their implementation. For example, a JavaScript unit test runner will require both the `IUnitTestingZone` and the `ILanguageJavaScriptZone`.

```cs
namespace Foo.Bar.JavaScriptTestRunner
{
  [ZoneMarker]
  public class ZoneMarker : IRequire<IUnitTestingZone>, IRequire<ILanguageJavaScriptZone>
  {
  }
}
```

Creating a zone marker with multiple dependencies can be considered an "anonymous" or "implicit" zone, made up of the listed dependencies, rather than explicitly defined with a zone definition class. Typically, this should only be used when the components that make up the zone are only to be used for end user facing features. There is no need for an explicit zone definition, because nobody needs to depend on these components and therefore include the zone in an `IRequire<TZone>`.

However, if the components are expected to be used by another zone (that is, the feature is an internal, code facing feature) then a zone definition should be created that encapsulates the dependencies. The zone marker should then only require this single, new zone definition, and no others. This gives depending code a zone definition it can require. Failure to do this means error prone repetition of zone dependencies in the dependent zones.

Put another way, consider the dependencies of a zone to be an implementation detail. Do not require depending code to know your dependencies in order to consume your components - provide a zone definition.

### Class zone marker

The `ZoneMarker` class and `[ZoneMarker]` attribute apply to all types in a namespace. It is possible to apply a single class to a zone, by applying the `[ZoneMarker]` attribute directly to a component class. Usage is exactly the same, either specify dependencies as parameters to the `[ZoneMarker]` constructor, or implement `IRequire<TZone>` directly on the component for each dependency.

Generally speaking, most registration will use `ZoneMarker` classes to apply a zone to all components in a namespace.

### Empty zone marker

A namespace can be marked with an empty zone marker, meaning it is zone aware, but does not require any other zone.

```cs
namespace Foo.Bar
{
  [ZoneMarker]
  public class ZoneMarker
  {
  }
}
```

All components in the `Foo.Bar` namespace and below are now zone aware, but do not require any other zones to be active. The component model filter will always allow these components through the zone filter. They are assumed to be infrastructure components, that is, essential to the correct running of the product, and always required.

### Example

When creating the component containers, the Component Model applies the zone filter to every part, and checks that each part belongs to one or more active zone. It will walk the namespace segments of the type name of the part, and check for zones at each level. If there are any zones attached to that namespace, then all zones must be active, or the part is not included in the container. The filter will walk every segment in the namespace, and all zones in all namespace segments must be active, or the part is not included. If there are no zones specified for the part, it is not included.

```cs
// The namespace of the zone definition is irrelevant
[ZoneDefinition]
public interface IMyZone : IDependentZone
{
}

namespace Foo.Bar
{
  [ZoneMarker]
  public class ZoneMarker : IZone, IRequire<IMyZone>
  {
  }
}

namespace Foo.Bar.Baz
{
  [ShellComponent]
  public class MyComponent
  {
  }
}
```

Given the code above, when evaluating the `MyComponent` part type, the filter will:

* Look for any zones at the `Foo` namespace. None are defined, so continue.
* Look for any zones at the `Foo.Bar` namespace. The `Foo.Bar.ZoneMarker` class has applied the `IMyZone` zone to this namespace.
  * The `Foo.Bar.ZoneMarker` class has applied the `IMyZone` zone to this namespace.
  * The transitive dependency `IDependentZone` is automatically also applied to this namespace.
  * If `IMyZone` is not active, the part is immediately rejected, and `MyComponent` is not included in the container.
  * Perform the same check for `IDependentZone`. It must be active, or the component is rejected.
* Look for any zones in the `Foo.Bar.Baz` namespace. None are defined, so continue.
* There are no more namespace segments. Because zones were encountered (and since the component hasn't already been rejected), the part is included in the container. If no zones were encountered, the part is rejected.

