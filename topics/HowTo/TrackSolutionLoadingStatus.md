[//]: # (title: Track Solution Loading Status)

**What you should know beforehand:**
* [Component model](ObtainComponentsInRuntime.md)
* [Lifetime](WorkWithLifetime.md)
* [Signals](WorkWithSignals.md)

**Examples ([?](HowTo_HowTo.md#sample-solution)):**
* [SolutionStateTracker.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/SolutionStateTracker/SolutionStateTracker.cs)

Sometimes, it is necessary for the plugin to begin its work only after the solution is completely loaded (or right before it is closed). Below you'll find one of the possible implementations of a shell component that tracks solution loading status.
 
This example also demonstrates how the [ReSharper component model](ObtainComponentsInRuntime.md) can be used. Here the task is solved using one shell component and one solution component. For details, see the notes below.

```
[ShellComponent]
    public class SolutionStateTracker : ISolutionStateTracker
    {
        public ISolution Solution { get; private set; }        
        public ISignal<ISolution> AfterSolutionOpened { get; }
        public ISignal<ISolution> BeforeSolutionClosed { get; }
 
        public SolutionStateTracker([NotNull] Lifetime lifetime)
        {            
            AfterSolutionOpened = new Signal<ISolution>(lifetime, "SolutionStateTracker.AfterSolutionOpened");
            BeforeSolutionClosed = new Signal<ISolution>(lifetime, "SolutionStateTracker.BeforeSolutionClosed");            
        }
 
        private void HandleSolutionOpened(ISolution solution)
        {
            Solution = solution;
            AfterSolutionOpened.Fire(solution);
        }        
 
        private void HandleSolutionClosed()
        {
            if (Solution == null)                
                return;

            BeforeSolutionClosed.Fire(Solution);
            Solution = null;
        }       
 
 
        [SolutionComponent]
        private class SolutionStateNotifier
        {
            public SolutionStateNotifier([NotNull] Lifetime lifetime,
                [NotNull] ISolution solution,
                [NotNull] ISolutionLoadTasksScheduler scheduler,
                [NotNull] SolutionStateTracker solutionStateTracker)
            {
                if (lifetime == null)
                    throw new ArgumentNullException("lifetime");
                if (solution == null)
                    throw new ArgumentNullException("solution");                
                if (scheduler == null)
                    throw new ArgumentNullException("scheduler");
                if (solutionStateTracker == null)
                    throw new ArgumentNullException("solutionStateTracker");                
 
                scheduler.EnqueueTask(new SolutionLoadTask("SolutionStateTracker",
                    SolutionLoadTaskKinds.Done, () => solutionStateTracker.HandleSolutionOpened(solution)));                
 
                lifetime.AddAction(solutionStateTracker.HandleSolutionClosed);
            }
        }
    }
```

## Notes

* `SolutionStateTracker` is a shell component (marked with the `[ShellComponent]` attribute), so it is created right after ReSharper starts and has the same lifetime with ReSharper.
* `SolutionStateNotifier` is a solution component (marked with the `[SolutionComponent]` attribute), so it is created only when a solution is opened in Visual Studio and has the same lifetime with the solution. 
* `SolutionStateNotifier` tracks solution status using the `ISolutionLoadTasksScheduler` object demanded via the constructor argument.
* Once the solution is loaded, `SolutionStateNotifier` calls the particular method of the injected `SolutionStateTracker`, which, in turn, fires a signal.
* `SolutionStateNotfier` also tracks solution closing:
    ```
    lifetime.AddAction(solutionStateTracker.HandleSolutionClosed);
    ```
* The `Lifetime.AddAction` method schedules an activity upon lifetime termination. As the lifetime for the `SolutionStateNotifier` is the same as the solution lifetime, the signal is called once the solution is going to close.
* To use `SolutionStateTracker`, you should simply subscribe to a corresponding signal using the `Advise` method:
    ```
    solutionStateTracker.AfterSolutionOpened.Advise(lifetime, () => {do somehting...});
    ```
    To obtain a `SolutionStateTracker` instance, you can, for example, get it from the current context (e.g., if you use it in an action):
    ```
    var solutionStateTracker = context.GetComponent<SolutionStateTracker>();
	solutionStateTracker.AfterSolutionOpened.Advise(lifetime, () => {do somehting...});
	```
	or demand it via the constructor argument of your component:
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