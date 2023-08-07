[//]: # (title: Containers, Parts and Catalogues)

 >  This is an advanced, deep dive into the implementation of the Component Model. It is not necessary to understand this in order to use the Component Model, or solution or shell components.
 >
 {type="note"}



The Component Model is designed to create loosely coupled, composable applications. Shell and solution components are just one example of doing this, and are implemented on top of infrastructure that allows for more flexibility and custom handling of component creation and lifetime. In other words, the Component Model is a complete [Inversion of Control](http://en.wikipedia.org/wiki/Inversion_of_control) framework.

At application startup, ReSharper creates several "containers". These are the environment container, the shell container and, when a solution is opened, the solution container. These containers are created based on a catalogue set, that is, a collection of catalogues, where a catalogue is a collection of assemblies and exported types known as "parts". By default, a part is recognised as a type that is decorated with an attribute that derives from `PartAttribute`. Both `ShellComponentAttribute` and `SolutionComponentAttribute` derive (indirectly) from `PartAttribute`.

The container is responsible for creating object instances from the part types. Any dependencies declared in the part type's constructor are also created and passed in. The container is also responsible for lifetime management of these parts, terminating each part's `Lifetime` or calling `Dispose` when the container itself is terminated. All object instances are created when the container is composed.

Typically, this infrastructure is not required by end-user code. Marking code as a shell or solution component is usually enough. However, it is sometimes necessary to manage lifetime or construction in a different way, and this can be done with a separate container. For example, the PSI's `DaemonStageManager` class creates its own container to manage the classes that handle analysis highlighting, in order to properly handle the correct grouping and ordering of the stages. It creates the container based on the application's catalogue set, filtering for parts that are decorated with the `[DaemonStage]` attribute. It is declared as a `[SolutionComponent]`, which means the container and all of its components are also scoped to the solution lifetime.

## Containers

The `ComponentContainer` class is the implementation of the container, and is responsible for resolving parts into object instances. It takes a collection of `IComponentDescriptor` instances, each of which know how to create a single part. When the container is "composed", the component descriptors are iterated, and the component is created, resolving dependencies as it does.

Once composed, the container can resolve part type requests by passing the type to a list of `IValueResolvers` that can map a part type request to one or more component descriptor instances. Once the descriptors have been resolved, the descriptor's cached object instance is used. The value resolvers job is to allow for creating new descriptors for types that aren't among the component descriptors, for example, `IEnumerable<TPart>`, `IViewable<TPart>` or `Lazy<TPart>`.

Creating a container is simply a matter of calling `new`, passing in a `Lifetime` and an id for diagnostics. The `Lifetime` is the controlling lifetime of the container - when this `Lifetime` terminates, the container terminates, as do all component descriptors and value resolvers. As a part of this, all component instances are also terminated, and any `Dispose` methods are called.

The third parameter to the constructor is an instance of `IInitializationStrategy`. This is used to control how the parts are created when the container is composed. More on this later.

## Registering part types

Before the container can be composed, it must know how it will create the component instances for the requested parts. This is handled by a collection of `IComponentDescriptor` instances, each of which describes how to create a single part. These component descriptors are passed to the container by one of the following methods:

* `RegisterSource(IViewable<IEnumerable<IComponentDescriptor>> componentSource)` registers an observable collection of sequences of component descriptors. This observable collection can add or remove new descriptor collections at any time.
* `RegisterDescriptors(IList<IComponentDescriptor> descriptors)` and `RegisterDescriptors(Lifetime, IList<IComponentDescriptor> descriptors)`. The latter allows for descriptors to be tied to a `Lifetime`, and  removed when the `Lifetime` is terminated.

The Component Model provides several implementations of `IComponentDescriptor`, including parts created as singletons, parts created by factory methods, and parts wrapped in `Lazy<T>`. Rather than use the `RegisterDescriptors` methods directly, it is usually easier to use the extension methods on `ComponentContainerEx`:

* `Register(this ComponentContainer, Type type)`, `Register<T>(this ComponentContainer)` and `Register<T>(this ComponentContainer, Lifetime)` register a type with the container, optionally with a `Lifetime` to allow for removing the type from the container. The descriptor added to the container is an instance of `SingletonTypeComponentDescriptor`, which creates a singleton instance of the type.
* `Register(this ComponentContainer, object instance)` registers an existing object instance.
* `Register(this ComponentContainer, Func<IValueResolveContext, T> factory)` registers a factory that takes in a resolve context, and returns an object. The returned value is treated as a singleton.

However, the simplest way of providing descriptors is to use the `RegisterSource` method with an instance of `CatalogueComponentSource`. This class converts a catalogue set implementing `IPartsCatalogueSet` into a collection of component descriptors. Since it's also an `IViewable` of component descriptor collections, whenever a new catalogue is added to the catalogue set, it broadcasts a new collection of component descriptors.

The source is also created with an `IPartsSelector` and a collection of `IPartsCatalogueFilter` instances. Both items filter the parts from the catalogue set, but for different purposes.

The `IPartsSelector` is used to select which parts can be created. Typically this is an instance on the static `PartsSelector` class:

* `PartsSelector.Leafs` selects only leaf parts. This allows parts to be overridden by deriving classes. If the base class and the derived class are both identified as a part (typically due to a `[Part]` derived attribute), then both classes would be eligible when resolving a base interface. Selecting only leaf parts will strip out the base classes, and only select the derived.
* `PartsSelector.LeafsAndHides` selects only leaf parts, but also allows a part to replace another part. If a part implements `IHideImplementation<TPart>` then the part with type `TPart` is not selected. This allows for overriding implementations without deriving.
* `PartsSelector.All` selects all parts, leaf and non-leaf. This can cause the Component Model to throw if a requested interface can be resolved by both a base and derived component instance.
* `PartsSelector.Default`. The same as `PartsSelector.LeafsAndHides`.

The collection of `IPartsCatalogueFilter` can be used to filter the parts that are to be created, for example, using the `CatalogueAttributeFilter` to filter for `SolutionComponentAttribute` parts.

The easiest way to register a source is to use one of the `RegisterCatalogue` extension methods from `CatalogueComponents`:

```csharp
public void Register(ComponentContainer container, IPartsCatalogSet catalogSet)
{
  // Register a catalog set with an attribute filter for MyPartAttribute
  container.RegisterCatalogue<MyPartAttribute>(catalogSet);
}
```

Note that if a new component descriptor is added after the container has been composed, the container is recomposed, and the new parts are immediately instantiated and available.

## Chaining containers

The container can be "chained" to another container. When resolving a type, if it can't be found, the chained container will be called. This allows for satisfying dependencies that exist outside of the current container. For example, the solution container has `SolutionAttribute` based parts registered, but some of those solution components will require dependencies that are shell components. By chaining the solution container to the shell container, those dependencies can be satisfied from the shell container.

Chaining is handled with the `ChainTo` extension method:

```csharp
public ComponentContainer Create(CatalogComponentSource source, ComponentContainer parent)
{
  return new ComponentContainer(lifetime, "myContainer")
    .RegisterSource(source)
    .ChainTo(parent);
}
```

## Registering resolvers

Each part type is registered with the container as an instance of `IComponentDescriptor`. This interface inherits the `IValueDescriptor` interface, which provides the `GetValue` method for getting the actual object instance of the descriptor.

Each registered descriptor can create an instance of a specific type of object, but there are no descriptors for returning a list of parts that implement a common interface - for example, if a component wants to get all object instances that can handle code completion, it can add a constructor dependency of `IEnumerable<ICodeCompletionItemsProvider>`, or to get a live list that handles container recompisition, `IViewable<ICodeCompletionItemsProvider>`.

Requests for these types are handled by an extensible set of value resolvers, implementing the `IValueResolver` interface. These value resolvers take in a request for a part type, and return an instance of `IValueDescriptor`. For example, the default component resolver simply looks up the part type in the registered component descriptors, and returns it.

When resolving, the container will chain the request for a type through a known set of resolvers:

* `EnumerableValueResolver` which will return an `IEnumerable<TPart>` with all component instances of type `TPart`
* `ViewableValueResolver` returns a live `IViewable<TPart>` which will include any new component instances found when the container is recomposed (perhaps a new catalogue has been added to a catalogue set).
* The default component resolver, which simply looks up the part in the list of component descriptors.
* `DelegatingContainerValueResolver` will handle any requests for `IComponentContainer` by looking at the current context and finding the parent container
* `LifetimeValueResolver` will satisfy any requests for `Lifetime` by creating a value resolver that will manage a `LifetimeDefinition`. When asked for the value, it will create a `Lifetime` instance. When the container is terminated, all value resolvers are terminated, and any `LifetimeDefinition` instances here will also be terminated. This is how the `Lifetime` passed to a component constructor is managed.
* `LoggerValueResolver` will resolve `ILogger` requests by deferring to `Logger.GetLogger()` using the fully qualified name of the requesting type.
* `LazyValueResolver` will resolve `Lazy<TPart>` requests. The returned `Lazy<T>` will lazily resolve the requested part when used. Note that this requires `JetBrains.Util.Lazy<T>` and does NOT support `System.Lazy<T>` (since ReSharper is a .net 3.5 application and `System.Lazy<T>` is a .net 4 type. ReSharper will throw an explicit exception is `System.Lazy<T>` is used.
* `OptionalValueResolver` will resolve `Optional<T>` requests. The target in the returned `Optional<T>` is immediately evaluated, but can return null without throwing exceptions.

The `ComponentContainer.RegisterResolver` method allows for registering custom resolvers that get inserted, and is how container chaining is implemented.

## Composition

Composition is simply handled by a simple call to `Compose`. The container will pass each `IComponentDescriptor` to the `IInitializationStrategy.Schedule` method of the strategy passed in to the container constructor. If no strategy was passed, the `InitializationStrategyDefault` class is used, which immediately calls `GetValue()` on the descriptor, which immediately creates and caches the component instance.

Alternatively, when creating the container, you can pass in an instance of `DelayedInitializationStrategy`, which schedules each call to `IComponentDescriptor` to the current `Dispatcher`, as a background priority action. This means it will get called when the application's `Dispatcher` is next idle. Note that initialisation will still happen on the current thread.

 >  This strategy is very useful for application start up, as it means the application no longer blocks during startup. However, it should be used with care! Using the default strategy, all objects are instantiated before the `Compose` method completes. With `DelaredInitializationStrategy`, the call to `Compose` completes before all objects are instantiated.
>
> However, trying to resolve a part that is still queued up for delayed initialisation simply creates and caches the object instance, resolving any dependencies as it does so. It is therefore safe to use components with delayed initialisation.
 >
 {type="note"}

A component can opt-out of delayed initialisation by setting the `Requirement` property on its attribute:

```csharp
[ShellComponent(Requirement = InstantiationRequirement.Immediate)]
public class MyImmediateComponent
{
}
```

This feature has been added for application startup, and is automatically applied to the shell container, but not to other containers. As such, the `Requirement` property is only implemented on the `ComponentAttribute`, rather than the `PartsAttribute`. However, (warning - implementation detail!) the reflection code to look for the `Requirement` property is loosely typed, and will work with a property declared on a `PartsAttribute` derived type.

## Resolving instances

Once the container has been composed, object instances can be retrieved using the extension methods in `ComponentContainerEx`.  These methods include:

* `GetViewable<T>` returns an `IViewable<T>` observable collection of all current instances of `T` in the container. It will also notify any new instances that are added to the container from a component source.
* `GetComponents<T>` returns an `IEnumerable<T>` of all current instances of `T` in the container. This is not a "live" collection - any new instances added are not reflected in the enumerable.
* `GetComponent<T>` returns a single instance of `T`. If there are more than one instances, this method will throw.
* `TryGetComponent<T>` tries to return an instance of `T`, if it exists in the container. If it doesn't exist, the out parameter is null, and the method returns `false`.
* `HasComponent<T>` looks to see if an instance of `T` exists in the container.

These methods take some of the legwork out of managing an instance of `IValueResolveContext` when calling `ComponentContainer.Resolve` to get an instance of `IValueDescriptor`. If a descriptor is returned (the container will return `null` if a descriptor is not available), the `IValueDescriptor.GetValue` method is called, returning the value. Typically, the value was already been created when the container was composed, and the cached value is returned.

The `T` passed to these methods can be any type in the type hierarchy of the required instance. It can be the most derived class, an abstract base class or an implemented interface.

## Cardinality

Typically, a container will be made up of only the most derived types of a hierarchy, thanks to the use of the `LeafsAndHides` parts selector. However, this does not dictate how many instances of a particular base type or interface are available in the container. For example, given the following:

```csharp
public interface IFoo { }
public class Foo1 : IFoo { }
public class Foo2 : IFoo { }

public class Base { }
public class Derived { }
public class MostDerived { }
```

The container will make available two instances of `IFoo` - `Foo1` and `Foo2`. However, there is only implementation of `Base` - the `MostDerived` class.

It is important to get this right when consuming dependencies in the constructor. For example, the following constructor will fail, since there are multiple instances of `IFoo` that can satisfy the parameter:

```csharp
[ShellComponent]
public class MyComponent(IFoo foo)
{
}
```

Each component descriptor is assigned a "cardinality" - single or multiple. This cardinality is set the first time the component is resolved. If a single instance is being resolved, such as the use of `IFoo` in the example above, the cardinality is set to "single". If an `IEnumerable<T>` or `IViewable<T>` is being resolved, the cardinality of `T` is set to "multiple".

When running a checked build, the cardinality is verified whenever a component is resolved, and if the cardinality of the request does not match the cardinality of the component, an error is logged, and an exception is thrown. When not running a checked build, this validation does not occur and the Component Model will try to resolve the component. This will succeed when requesting multiple instances of a component with single cardinality (it will return a collection of one item), however it will definitely fail when trying to resolve a single instance of a component with multiple cardinality (which should it return?).

## Overriding and replacing components

As seen above, only the most derived instance of a type is added to the container. This makes it possible for third party code to override types belonging to the ReSharper Platform. While this can be useful, it is discouraged, as only one third party can override a component - if a second third party tries to override the same component, then multiple leaf instances are introduced to the component, and resolution will fail.

It is also possible to replace a component completely, without deriving from it. If a component implements `IHideImplementation<T>`, then the `LeafsAndHides` selector will replace the existing implementation `T` in the container with the current component.

```csharp
[ShellComponent]
public void AnotherComponent : IFoo
{
}

[ShellComponent]
public void MyComponent : IFoo, IHideImplementation<AnotherComponent>
{
}
```

While both `AnotherComponent` and `MyComponent` implement `IFoo`, only `MyComponent` is added to the container, since it explicitly declares that it hides the implementation `AnotherComponent`. Note that it hides a specific implementation class, and it does not hide all implementations of `T`.

## Catalogue sets

The main source of part types for a container is a catalogue set, which is essentially an observable collection of catalogues, implementing the `IPartsCatalogueSet` interface.

```csharp
public interface IPartsCatalogueSet
{
  IViewable<IPartsCatalogue> Catalogues { get; }
  IViewable<IPartsCatalogue> CataloguesPreview { get; }
}
```

The observable collection `Catalogues` is the list of current catalogues known to the catalogue set, but it will also notify subscribers of new catalogues being added to the set. This allows for recomposition of the catalogue set, and therefore any containers that are based on this catalogue set.

The `CataloguesPreview` property is a similar observable collection to the `Catalogues` property, except new catalogues are added to this collection first. This allows for viewers to be notified of a new catalogue before it is processed, allowing for work before a container processes the new types.

At application startup, the `JetEnvironment` class is responsible for setting up the catalogue sets. It creates an instance of `FullProductCatalogueSet`, passing in a list of all of the assemblies that make up the product. The catalogue set is initially created with a single catalogue made up of all of these assemblies, and containing a list of types that are decorated with `[Part]` derived attributes. This catalogue set is an unfiltered collection of all parts that are available to the application. Note that no component instances have been created yet.

It also exposes an `EnvironmentPartCatalogSet`, which is an instance of `PartsCatalogueSet`. This is a filtered view of the `FullProductCatalogSet` that strips out any parts that are not part of a [Zone](Platform_Zones.md). This catalogue set is used to populate the component containers.

## Catalogues

An `IPartsCatalogue` is a class that maintains and exposes a collection of metadata about the assemblies in the catalogue, and all the types that are to be treated as parts. By default, the parts are defined as any type that is decorated with an attribute that derives from `PartAttribute`, but this isn't required, and the Component Model can use any type. However, ReSharper's containers only support `[Part]` decorated parts.

The metadata about the assemblies and the types does not come from reflection, but is efficiently read directly from the IL of the assemblies, and cached.

The default implementation of `PartsCatalogue` maintains a simple list of assemblies and types, while the `AttributedIndexedPartsCatalogue` optimises to parts that are defined by attribute, and indexes each part type by attribute, allowing quicker access to all parts that are e.g. `[SolutionComponent]` parts.
