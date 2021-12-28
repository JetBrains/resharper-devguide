[//]: # (title: Code Completion)

Code completion is handled by ReSharper, as long as the reference also implements the `ICompleteableReference` interface. This interface inherits from `IReference` and defines just one method:

```csharp
public interface ICompleteableReference : IReference
{
  ISymbolTable GetCompletionSymbolTable();
}
```

This method should return a symbol table of candidates. Each candidate will be added to the code completion list. The implementation for this is usually very straightforward:

```csharp
public ISymbolTable GetCompletionSymbolTable()
{
  return GetReferenceSymbolTable(false);
}
```

We simply return the symbol table from `IReference.GetReferenceSymbolTable` by passing `false` for `useReferenceName`. This is already our list of candidates, so we don't need to do any further processing.