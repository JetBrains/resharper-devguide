---
---

# Lifetime Management

A lifetime is made up of two parts, the `Lifetime` instance, which is passed around and used to register callbacks, and a `LifetimeDefinition` which is used to manage the lifetime.

## LifetimeDefinition

You cannot terminate a `Lifetime` instance; unless you create it, you do not own it, therefore you shouldn't terminate it. Instead, when you create a `Lifetime`, you receive an instance of `LifetimeDefinition`, and this class allows you to terminate the `Lifetime`. Here's what it looks like:

```csharp
public class LifetimeDefinition
{
  public void Terminate();
  public bool IsTerminated { get; }
  public Lifetime Lifetime { get; }
}
```

It is a very simple class. You can get the `Lifetime` instance it is associated with, and calling `Terminate` will terminate this instance. The `IsTerminated` property will return true if `Terminate` has been called.

When terminating, the callbacks registered to the `Lifetime` are called in reverse order of registration. That is, most recently registered first. All callbacks are called, even if one throws an exception - the exception is logged, and then ignored (favouring continuing cleanup over exception reporting).

## Lifetime Creation

You can use the `Lifetimes` class to create your own `Lifetime` instances.

> **NOTE** You should normally let the Component Model [create a `Lifetime` for your component](ComponentModel.md).

### Short lived Lifetimes

If you only need a `Lifetime` for a short period of time, the `Lifetimes.Using(Action<Lifetime> f)` method will create a new instance, call your method and then immediately terminate the `Lifetime`. There is also an overload that takes in a `Func<Lifetime, TRetval>` to allow your method to return a value.

### Nested Lifetimes

If you want a longer-lived `Lifetime`, you can call `Lifetimes.Define`:

```csharp
Lifetimes.Define(Lifetime lifetime, string id, Action<LifetimeDefinition, Lifetime> action, ILogger logger);
```

This method takes in a `Lifetime` to act as a parent; the new `Lifetime` is a "nested" lifetime. Terminating the parent `Lifetime` will also terminate this new child `Lifetime`. Of course, if the child terminates first, it removes itself from the parent's cleanup.

> **NOTE** If you don't have a parent `Lifetime` available, you can use `EternalLifetime.Instance`. Generally speaking, `EternalLifetime` should be avoided, as it is never terminated. Items scheduled against `EternalLifetime` will not run, and will not get garbage collected, effectively resulting in a memory leak. The preferred use case for `EternalLifetime` is to act as a parent for another `Lifetime`.

The `Define` method has a number of optional parameters, firstly an id, which is only used for diagnostic purposes. Secondly, an action that is called as soon as the `LifetimeDefinition` has been created. And finally, an optional instance of `ILogger`. If no logger is passed in, the default logger instance from `Logger.Interface` is used (see the Logging section for more details on logging).

