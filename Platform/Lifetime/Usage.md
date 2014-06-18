# Usage

The `Lifetime` class looks like this:

```cs
public class Lifetime
{
  public Lifetime AddAction(Action f);
  public Lifetime AddBracket(Action fOpening, Action fClosing);
  public Lifetime AddDispose(IDisposable item);
  public Lifetime AddRef(object o);
  public bool IsTerminated { get; }
}
```

Your code receives a `Lifetime` instance, and registers its cleanup callback using `AddAction`. It then doesn't need to do anything else. When the `Lifetime` is terminated, the callback is called, and the action is performed. Actions are called in reverse order to how they were added. If an exception is thrown by an action, it is logged, and the next action is called.

```cs
public void MyMethod(Lifetime lifetime)
{
  lifetime.AddAction(() => { /* Do cleanup */ });
}
```

* The `AddBracket` method accepts two callbacks. The first, `fOpening`, is called immediately, and the second, `fClosing` is registered to run when the `Lifetime` is terminated. This way, the methods form a "bracket" around the duration of the `Lifetime` object.

* There is also a bridge between `IDisposable` and `Lifetime`. The `AddDispose` method simply registers a callback that will call `Dispose` on the given `IDisposable`.

* The `AddRef` method keeps a given object alive (i.e. it won't be garbage collected) until the `Lifetime` is terminated.

* You can check to see if a `Lifetime` has been terminated by calling the `IsTerminated` property. Note that you can't terminate a `Lifetime` directly.

## Extension methods

Generally speaking, especially when writing plugins, you are more likely to pass a `Lifetime` to a method than to directly add your own cleanup callbacks. When consuming services, adding items or registering callbacks, passing in a `Lifetime` will allow that service or object to add the cleanup, removal or un-registration code for you. You should generally favour method overloads that take a `Lifetime` over those that don't.

For example, the `JetBrains.Util.CollectionUtil` class provides several extension methods for `ICollection<T>`. These methods simply call `Lifetime.AddBracket` to immediately add the item, and will remove the item when the `Lifetime` is terminated. This means the items only exist in the collection for the duration of the `Lifetime`.

Similarly, the `Threading` subsystem's `JetDispatcher` class, which can be used to dispatch actions to the main thread has an overload to the `BeginInvoke` method which take a `Lifetime`. This queues the given action to execute on the main thread, at some point in the near future. If you terminate the `Lifetime`, the action is (effectively) removed from the queue, and the action isn't executed. This is an easy way to prevent race conditions. 

