[//]: # (title: Component Model)



ReSharper has a very composable architecture, which allows for a loosely coupled, easily extensible design. Functionality is implemented in terms of components that advertise themselves to the Component Model, which in turn is responsible for lifetime management, as well as wiring up the inter-dependencies of the components. The Component Model will look for classes marked with specific attributes, which declare the lifetime scope of the component. Dependencies are declared as constructor arguments. At the appropriate time, the Component Model will create new instances of the components, ensuring all dependencies are created first, and passed into the constructor.

 >  If you are familiar with the concept of [Inversion of Control](http://en.wikipedia.org/wiki/Inversion_of_control) and [Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection), ReSharper's Component Model follows these patterns, and implements an IoC Container to automatically create and wire up dependendencies
 >
 {type="note"}

This loosely coupled design allows for easily extending ReSharper - new components can easily be advertised to the Component Model, and services are available for consumption by declaring constructor arguments. It is not possible to integrate with ReSharper without using and understanding the Component Model.

Conceptually, the Component Model is very similar to Microsoft's [Managed Extensibility Framework (MEF)](http://msdn.microsoft.com/en-us/library/dd460648.aspx) although there are some fundamental differences, such as the ability to live load and unload new components, during runtime.

ReSharper defines many component types. The two most interesting are `ShellComponent` and `SolutionComponent`. More details about other component types can be found in the discussion about `Parts` in [Containers, Parts and Catalogues](ContainersPartsCatalogues.md).

## Shell Components

A **Shell Component** is a class that gets created when ReSharper starts. Shell components are typically marked with the `[ShellComponent]` attribute. It has essentially the same lifetime as ReSharper itself. Shell components are effectively singletons, and are most useful when creating a service that is not solution specific. For example, Live Template macro definitions are shell components. They do not use or depend on anything related to a solution, and do not change for the lifetime of the shell.

To define a shell component, simply write:

```csharp
[ShellComponent]
public class MyClass
{
  // ...
}
```

ReSharper will create an instance of this class when the shell starts up.

If a component needs to talk to other components, it can simply add them as a constructor argument, and the Component Model will ensure that the dependent component is created first, and passed in on construction. The dependent component must have the same lifetime scope or longer, so a shell component can only request another shell component.

```csharp
[ShellComponent]
public class SomeOtherClass
{
  private MyClass myClass;

  public SomeOtherClass(MyClass myClass)
  {
    this.myClass = myClass;
  }
}
```

Constructor injection is the preferred means of accessing dependencies. However, there are some cases where this isn't possible - for example, Action handlers aren't created by the Component Model. In this case, you can use the Service Locator pattern and ask for the dependency:

```csharp
public void SomeMethod()
{
  // Shell.Instance is a static property
  var myClass = Shell.Instance.GetComponent<MyClass>();
}
```

Note, however, that `Shell.Instance.GetComponent` will only return shell components, not solution components.

## Solution Components

**Solution Components** are components that are tied to the lifetime of a solution. They are created each time a new solution is opened, and use the `[SolutionComponent]` attribute. Again, constructor injection is the preferred means of satisfying dependencies. A `SolutionComponent` can depend on both `SolutionComponent` and `ShellComponent` instances, because shell components have a longer lifetime than solution components.

You can also use the Service Locator pattern to ask for solution components. There are several ways of getting at the solution level Component Model, usually with a `GetComponent` or `GetComponents` extension method. For example, the `IDataContext` object passed to action handlers can be used to retrieve components, or the `ISolution` and `IProject` interfaces have extension methods. Again, when asking for a component directly, you can ask for a solution component or shell component. For example:

```csharp
var solution = context.GetData(JetBrains.ProjectModel.DataContext.DataConstants.SOLUTION);
var dtm = solution.GetComponent<DocumentTransactionManager>();
```

One very important object that can be injected or asked for is `ISolution`. This allows a solution level component to get at its current context - the solution it's been created for.

## Component cleanup

A component is tied to a particular lifetime - the lifetime of the shell for `ShellComponent` instances and the lifetime of the solution for `SolutionComponent` instances. When the solution is closed, or the shell terminates, the component is now out of scope, and is available for garbage collection. If you need to do any explicit cleanup that can't be handled by garbage collection, you can implement `IDisposable` and the Component Model will call `Dispose` for you.

Alternatively, you should inject an instance of `Lifetime` into your constructor, and register callbacks with it. The `Lifetime` object is terminated at the end of the lifetime scope of the component, and will call any registered callbacks at that time. `Lifetime` is a very powerful construct for lifetime management, and is [documented separately](Lifetime.md).

You should ensure that you are not holding any references that are outside the scope of your component. For example, a shell component should not acquire and cache an instance of `ISolution`, or any other solution component, as this will cause memory leaks.

## Dependency failures

One downside to wiring up an application through the Component Model is that errors can cascade. For example, a shell component with a constructor that takes a solution component cannot be created, and the Component Model will throw an exception. But a second shell component that depends on this failed first shell component will now fail to be created, and so on, potentially destabilising the entire application. Care should be taken to require dependencies appropriately, and not cause exceptions in constructors.

Troubleshooting can initially seem very difficult, due to the cascading errors, all of which will throw exceptions. The important exception is the first one, which will tell you which component failed during creation. The failure will be related to one of the constructor parameters, so each parameter should be checked - is the parameter type a component? Does it come from the same or higher scope (i.e. is a shell component trying to depend on a solution component)? Does the parameter have the correct cardinality (single instance vs. collection)?

## Lazy and optional acquisition

Dependencies can be declared as lazy or optional.

A lazy dependency will only be resolved at the time of use, rather than at the time of creation of the owning component. Simply use `JetBrains.Util.Lazy<ComponentType>` instead of `ComponentType` when querying or injecting a component. Note that this does not work with the BCL's `System.Lazy` type, as this is a .net 4 type, and ReSharper is a .net 3.5 application (although it runs in .net 4 when hosted in VS2010 and VS2012).

An optional dependency is resolved at component creation time (same as normal components), but can be missing, and if so, will be null. You declare an optional dependency by requesting `Optional<ComponentType>` instead of `ComponentType`.

## Other component types

ReSharper supports several types of component, not just shell and solution components. For completeness, it also supports `EnvironmentComponent`, `ShellInstanceComponent` and `ProjectComponent`.

A `ShellComponent` has the same lifetime as ReSharper, but this might not be the same lifetime as Visual Studio - if someone suspends and resumes ReSharper, all shell components are released and then recreated. An environment component is created when ReSharper first starts up and lasts for the duration of the hosting application (such as Visual Studio), regardless of suspend and resume. It is intended for ReSharper's integration infrastructure, and generally speaking, should not be used by plugin authors.

`ShellInstanceComponent` is also an infrastructure component, created as the solution is opened. Again, there should be little need for it to be used by plugins, use `ShellComponent` instead.

`ProjectComponent` allows for a class to be created for each project that's loaded. This means, unlike shell and solution components, it's not a singleton - there will be one instance for each loaded project. If a project is unloaded or removed, the component is released. If a new project is added to the current solution, a new component is created. The Component Model can inject an instance of `IProject` to give the component some context. It's generally used for project level settings and properties (e.g. project level Live Templates, or C# language level). It is rarely used other than this.

### Derived component types

There are other component attribute types that derive from `ShellComponentAttribute` and `SolutionComponentAttribute` (e.g. `PsiComponentAttribute` and `CodeCleanupModuleAttribute`). These are treated the same as normal shell or solution components, except the derived attributes allow for grouping, or passing information to consumers. One other use is to mark the attribute with annotations to aid implementation. For example, any class with an attribute that has the `MeansImplicitUse` attribute will be marked as in use by ReSharper, even if `new` is not explicitly called on it. Similarly, a class with an attribute that has the `BaseTypeRequired` annotation will show a warning if that class does not derive from the named base type.

### Parts

Components such as shell and solution components, and derived types such as code cleanup module components are the most common usage of the Component Model in ReSharper, supporting known lifetime and usage requirements. However, they are not the only usage. The Component Model implements an Inversion of Control container, populated with "parts", and shell and solution components are implemented as nested components of parts identified by the `[ShellComponent]` and `[SolutionComponent]` attributes. Other components create their own containers for custom lifetime and usage handling. See the [Containers, Parts and Catalogues](ContainersPartsCatalogues.md) section for more details.
