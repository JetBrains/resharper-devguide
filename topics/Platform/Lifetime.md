[//]: # (title: Lifetime)

The `JetBrains.DataFlow.Lifetime` class is used for cleanup code and resource management. `Lifetime` is similar to, but different from `IDisposable`. In fact, you can consider `Lifetime` to be the "dual" of `IDisposable`.

 >  This class is used *extensively* throughout the codebase, and it is *highly recommended* that you understand what it is and how it works.
 >
 {type="note"}

`IDisposable` is best suited to deterministic cleanup of a limited number of objects in a small scope. That is, it's ideal when you have created one or maybe two objects that need cleanup at a specific point in code; the `using` keyword makes this particularly easy.

However, when dealing with multiple objects that require cleanup, or that live longer than a single method call, more and more code is required for housekeeping of the `IDisposable` instances, including providing the means to invoke `Dispose`. This pattern can quickly and easily spread to calling code, meaning a lot of extra code is required just to manage resources and cleanup.

The `Lifetime` class reduces this boilerplate by approaching the issue from the opposite direction. It is responsible for the housekeeping of the clean up tasks. Calling code can register callbacks with the `Lifetime`, and when it's terminated, those callbacks are executed, in reverse order, performing any required cleanup. This way, the housekeeping is moved out of user code.

Instead of returning an `IDisposable` object that knows how to perform cleanup, and which needs collating and disposing, a method that needs to perform cleanup will accept a `Lifetime` instance, and register a callback that performs the cleanup. Now a single instance is maintaining all of the cleanup code, and there is no resource management housekeeping code in the calling class.

Furthermore, the `Lifetime` class allows for "nested" `Lifetime` instances, which can either be terminated independently, or when parent `Lifetime` terminates.
