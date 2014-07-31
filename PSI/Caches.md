# Caches

You have no doubt seen ReSharper say it’s ‘loading caches’. These caches are collections that are kept in RAM so that when ReSharper needs to access them, they are available quickly. For example, R# has a declarations cache, which is basically a large set of data regarding the various declarations that R# has encountered while parsing code.

These caches are available to plugin developers. To get to the caches, all you need to get an instance of `IPsiServices` and get to the collection containing the caches you need..

Here's an example: say you want to get at all the type names that R# knows about. To do that, you get the instance of the `CacheManager`, call a `GetDeclarationsCache()` method to get the cache itself, and then simply call, e.g., `GetAllShortNames()` on the cache to get the names of all the CLR types that R# has encountered:

```cs
var services = provider.Solution.GetPsiServices();
var shortNames = services.Symbols.GetSymbolScope(LibrarySymbolScope.FULL, false,
  psiModule.GetContextFromModule()).GetAllShortNames();
```

The above returns an `IEnumerable<string>` that you can traverse to get all the names.

