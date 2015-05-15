---
---

# Managing reference providers

To build a reference provider, you implement `IReferenceProviderFactory`, `IReferenceFactory` and `IReference`. It is very easy to get these mixed up with the `ReferenceProviderFactory` class and the `IReferenceProvider` interface, however, they serve different purposes. `ReferenceProviderFactory` and `IReferenceProvider` are facades that manage the responsibility of delegating and aggregating the creation of custom references to multiple custom reference providers.

The `ReferenceProviderFactory` class hides the complexity of maintaining a large collection of custom reference providers. It is a `[PsiComponent]` that maintains an `IViewable<IReferenceProviderFactory>`. It has one public method:

```csharp
public class ReferenceProviderFactory
{
  public IReferenceProvider Create(IPsiSourceFile sourceFile, IFile file);
}
```

This method iterates over the collection of custom reference providers and calls `IReferenceProviderFactory.CreateFactory` on each, maintaining a list of the returned `IReferenceFactory` instances. It then returns a custom instance of `IReferenceProvider` that is based on this list of custom reference factories.

The `IReferenceProvider` interface is very similar to the `IReferenceFactory` interface:

```csharp
public interface IReferenceProvider
{
  ReferenceCollection GetReferences(ITreeNode element, ICollection<string> names);
  bool ContainsReference(ITreeNode element, IReference reference);
}

public interface IReferenceFactory
{
  IReference[] GetReferences(ITreeNode element, IReference[] oldReferences);
  bool HasReference(ITreeNode element, ICollection<string> names);
}
```

`IReferenceProvider` also acts as a façade, this time over the collection of `IReferenceFactory` instances, and in fact has a slightly different signature, more aimed at consuming references than creating them.

It is responsible for caching the references, and keeping them up to date when the PSI changes. If there is no data cached, or it is not up to date, it calls each of the `IReferenceFactory.GetReferences` implementations in turn, passing in the old references, if applicable. The returned new references are then cached and returned on demand. It will short circuit the cache update when possible by calling `IReferenceFactory.HasReferences`. If no reference factory has references here, the cache isn't consulted or updated. It also includes an optimisation that restores the previously known references if the current PSI transaction is rolled back.

There should be no reason to use `ReferenceProviderFactory` or `IReferenceProvider` directly, and is documented to prevent confusion between similarly named classes and interfaces.
