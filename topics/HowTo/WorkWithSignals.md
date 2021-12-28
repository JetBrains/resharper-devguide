[//]: # (title: Work with Signals)

**What you should know beforehand:**
* [Lifetime](WorkWithLifetime.md)

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [SignalEmitter.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Signals/SignalEmitter.cs)
* [SignalListener.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/Signals/SignalListener.cs)
* [SolutionStateTracker.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/SolutionStateTracker/SolutionStateTracker.cs) - more complex example of using signals

Signal is a way for a class to provide notifications. Signals are very similar to C# events, nevertheless, there are differences that make them more suitable for use within ReSharper data flow infrastructure.

```
public class SignalEmitter
{
    public ISignal<bool> SomethingHappened { get; }
 
    public SignalEmitter(Lifetime lifetime)
    {
        SomethingHappened = new Signal<bool>(lifetime, "SignalEmitter.SomethingHappened");
    }
 
    public void MakeItHappen(bool arg)
    {
        SomethingHappened.Fire(arg);
    }

}
 
 
public class SignalListener
{
    public SignalListener(Lifetime lifetime, SignalEmitter signalEmitter)
    {
        signalEmitter.SomethingHappened.Advise(lifetime, arg => MessageBox.ShowInfo($"Something happened and it's {arg}"));
    }
}
```

## Notes
* Signals must implement the `ISignal` interface.
* To "subscribe" a handler to a signal, use the `Advise` method. A signal allows any number of listeners.
* Unlike C# events, you should not care about "unsubscribing" from a signal to prevent memory leak. When you "subscribe" to a signal via `Advise`, you also pass a lifetime. Once the lifetime is terminated, ReSharper will take care about "unsubscribing" by itself.
* To fire the signal, use the `Fire` method.
* Signals have much in common with [IProperty](WorkWithIProperty.md).
* Signals are perfectly suited for MVC concept when an event should be fired by a view.
* In simple cases, when you need a signal only for notification purposes (with no payload), you can use the `ISimpleSignal` interface.