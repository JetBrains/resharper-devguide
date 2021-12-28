[//]: # (title: Advanced Lifetime Management)

Lifetime instances can be combined in very powerful ways.

## Intersection

The `Lifetimes.CreateIntersection2` method creates a `LifetimeDefinition` that is the *intersection* of a number of other `Lifetime`s passed in to the method. This means it will terminate when any one of the original `Lifetime` instances terminate. Of course, a `Lifetime` can only be terminated once, so nothing will happen when subsequent `Lifetime` instances terminate. The newly created intersection `LifetimeDefinition` can also be terminated independently, and when it is terminated, it doesn't affect any of the original `Lifetime` instances.

This is useful as a companion to `Lifetimes.Define`, which creates a `LifetimeDefinition` tied to a "parent" `Lifetime`. When the parent terminates, the child terminates, or the child can be terminated independently. `CreateIntersection2` creates a new `LifetimeDefinition` tied to a collection of "parent" `Lifetime` instances. When any of them terminate, the child terminates, or the child can be terminated independently. 

 >  The `CreateIntersection2` method will eventually replace the `CreateIntersection` method, which is currently marked as obsolete. 
>
> `CreateIntersection` has the same implementation as `CreateIntersection2`, but returns a `Lifetime`, while `CreateIntersection2` returns a `LifetimeDefinition`, which gives more control over the resulting `Lifetime`.
 >
 {type="note"}

## Synchronising Multiple Lifetimes

`Lifetimes.Synchronize` takes a collection of `LifetimeDefinition` and terminates them all when any one of them terminates.

## Sequential Lifetimes

The `SequentialLifetimes` class simplifies using consecutive, non-overlapping `Lifetime` instances, ensuring one is terminated before the next is created.

It maintains a private `LifetimeDefinition`, nested under a parent `Lifetime` provided in the constructor. When you call `Next` or `DefineNext`, the current `LifetimeDefinition` (if any) is terminated, a new one is created and stored, and the action you passed in is called, with the new `Lifetime`. (The action passed to `DefineNext` also takes the `LifetimeDefinition`). You can also terminate the current `LifetimeDefinition` explicitly by calling `TerminateCurrent`.

This removes the need for boilerplate management of a single `LifetimeDefinition` that needs to be created and terminated multiple times.