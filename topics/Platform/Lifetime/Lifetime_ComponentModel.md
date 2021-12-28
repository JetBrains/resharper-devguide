[//]: # (title: Component Lifetime)

The simplest way to get a `Lifetime` instance is to let the Component Model inject it into your constructor:

```csharp
[ShellComponent]
public class MyComponent
{
  public MyComponent(Lifetime lifetime)
  {
    // ...
  }
}
```

The Component Model will create a new `Lifetime` instance for each component, and is responsible for terminating it. The `Lifetime` is terminated when the appropriate component container is disposed. 

For example, the `Lifetime` for a component decorated with `[ShellComponent]` is terminated when the shell itself is terminated, normally when Visual Studio closes. A component decorated with `[SolutionComponent]` will have its `Lifetime` terminated when the currently open solution closes.