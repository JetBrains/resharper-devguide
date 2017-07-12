---
title: Work with Lifetime
---

**What you should know beforehand:**
* [Component model](/HowTo/ObtainComponentsInRuntime.md)

This section contains only brief details on what lifetime is. To learn more about lifetime, refer to the corresponding [chapter of this guide](/Platform/Lifetime.md).

Lifetime is one of the essential ReSharper API concepts. It's a pattern used in ReSharper to manage object's lifetime, e.g., dispose no longer needed objects. But why don't we use `IDisposable`? Long story short - `IDisposable` is best suited for deterministic clean up, when you have relatively small amount of objects that typically live only within a method (e.g., the `using` keyword). The fact that the resource management housekeeping code must be implemented in the calling class makes the `IDisposable` pattern not very good scalable. The Lifetime pattern solves the problem from the other side. Instead of returning an `IDisposable` object that knows how to perform cleanup, a method that needs to perform cleanup accepts a `Lifetime` instance, and registers a callback that performs the cleanup. Thus, only a single instance is maintaining all of the cleanup code. 

Let's see how it works in details.

## Obtaining a Lifetime
Typically, you should obtain the lifetime for your component from the ReSharper component model. As usual in the ReSharper API, you can get an instance of the `Lifetime` type using the constructor injection. 
```
[SolutionComponent]
public class MyClass
{
    public MyClass(Lifetime lifetime, ISolutionStateTracker solutionStateTracker)
    {
        solutionStateTracker.AfterSolutionOpened.Advise(lifetime, () => {do somehting...});
    }
}
```

As `MyClass` is a solution component, the lifetime will be terminated once the solution is closed in Visual Studio. You can use the lifetime instance to propagate this lifetime to other components. Note that many ReSharper API methods accept `Lifetime` as an argument. Even if you have an option not to provide it, you should always prefer a method overload that accepts the lifetime.

How this works? In the example above, we subscribe on the `AfterSolutionOpened` signal using `Advise`. The lifetime instance passed to the `Advise` method means that this subscription will live until lifetime is terminated (this will happen when the solution is closed). All routines related to unsubscription will be handled by the signal itself. So, typically, you will be passing a `Lifetime` instance to methods rather than writing your own cleanup code.  

## Running Your Own Cleanup Code
Nevertheless, you always have an opportunity to run your own cleanup code. To do this, you should register the cleanup callback using the `AddAction` method of the `Lifetime` type.
```
lifetime.AddAction(() => myObject = null);
```
Or, you can even use the `AddBracket` method that accepts two callbacks - one is run immediately and one after the lifetime is terminated. For example, adding and removing a UI element in a tool window can be bound to a particular lifetime:
```
lifetime.AddBracket(
    () => _containerControl.Controls.Add(_viewControl),
    () => _containerControl.Controls.Remove(_viewControl));
```
                        
## Custom Lifetime
Normally, you should let the component model to create a lifetime for your component. But, sometimes, it may be necessary to create your own short-lived lifetime. In this case, to define the lifetime, you should use the `Lifetimes.Define` method:
```
var myShortLifetimeDefinition = Lifetimes.Define(lifetime);
var myShortLifetime = myShortLifetimeDefinition.Lifetime;
```
To terminate your lifetime:
```
myShortLifetimeDefinition.Terminate();
```
### Notes
* `lifetime` is the parent lifetime. Once it is terminated, `myShortLifetime` is also terminated.
* `Lifetimes.Define` creates an instance of the `LifetimeDefinition` class which in turn allows you to manage the lifetime, e.g. terminate it.
