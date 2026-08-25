[//]: # (title: Caches)

ReSharper makes heavy use of caching in order to store all the information required to provide a semantic view of a codebase. The caches are calculated on solution load, stored to disk, and reloaded when the solution is subsequently loaded. Any time a file changes, the cache is notified, and can rescan the file and update its details.

The PSI implements the infrastructure for building and maintaining a cache, and provides a number of different caches, such as `ISymbolCache` - a symbol cache for looking up CLR `ITypeElement` declared elements, and `IWordIndex`, which is an index of words and the files they are used in (one use of this is reverse lookup of references).

The cache infrastructure is extensible, and other parts of the architecture create their own caches on top of this infrastructure. For example, the To Do Explorer maintains a cache that initially scans all files for to-do items, and keeps the list of items up to date as files change.

Caches are very useful for cross-file analysis, amongst other things. Normal analysis is performed in the context of a single file, and is called very frequently as the file is edited. It should therefore be as fast as possible, and not try to scan other files.

For example, imagine a scenario where an analyser is trying to ensure uniqueness of assembly level attributes. When a file is changed, the analyser can look in that file for the required attribute, but it would be very bad for performance if it would also try to scan all other files in the project for the same attribute. Instead, a cache can be implemented to initially scan for all attribute instances, and used by the analyser to look up potential duplicates.

The PSI caches can be retrieved by using `IPsiServices`, which provides direct access to `ISymbolCache` via `IPsiServices.Symbols`, or `IWordIndex` via `IPsiServices.WordIndex`. Other caches can be injected directly into a class constructor. 

A similar service is the `IPersistentIndexManager`, which is an extensible, persistent cache that is not related to file contents. For example, the unit test runner uses a persistent index to store results for each test. A persistent index is a disk backed key-value store (implemented on [LevelDB](http://en.wikipedia.org/wiki/LevelDB) although this is an implementation detail) that automatically manages what data is in memory or flushed to disk. The interface is a simple dictionary-like lookup.

Finally, the PSI maintains a set of in-memory caches. These are lightweight, transient caches that aren't persisted and are invalidated when any abstract syntax tree changes. They are intended to temporarily cache the result of operations that might occur again. Generally speaking, these caches are not intended for consumption from extensions, as they are used by other parts of the code. However, the cache can be retrieved either by injecting the component into a class constructor or by using `IPsiServices.Caches.GetPsiCache<T>` where `T` is the required cache, which implements `IPsiCache`, such as `InternalsVisibleToCache`.

## Cache versioning and invalidation

Because a persisted cache is stored to disk and reloaded on the next solution load, its on-disk data can outlive the code that produced it. If the serialized format or the meaning of the data changes between builds of a plugin, reloading an old cache into new code would deserialize stale or misinterpreted data. To guard against this, every persisted cache exposes a `Version`. The simplest way to write a persisted, per-file cache is to derive from `SimpleICache<T>`, which declares the version (the underlying `ICache` interface also has it):

```csharp
public abstract class SimpleICache<T> : IPsiSourceFileCache, ICacheWithVersion
{
  public virtual string Version => "1";
}
```

The version is part of the cache's on-disk identity. When the running code's `Version` differs from the version stored on disk, the persisted data is discarded and the cache is rebuilt from scratch. So whenever you change *what* a cache serializes or *how* you interpret it - a new field, a different meaning for an existing field, a changed hashing scheme - bump the `Version`. Forgetting to do so is a classic source of "works on a clean cache, breaks after reload" bugs, including Find Usages failing only after a solution is reopened.

A common convention is to keep a short note of each bump next to the property, and to compose the version from a manual number plus any external inputs that also affect the data. For example, `AnnotatedEntitiesCache` uses:

```csharp
public override string Version => $"3/{IndexableAnnotationsTable.AttributesHash}";
```

The leading `3` is bumped by hand when the format changes; the trailing hash invalidates the cache automatically when the external attribute set changes.

## Cache names, resolve live

Caches are most valuable for cross-file work, so it is tempting to store *resolved*, fully-typed, cross-file data - a resolved declared element, a pointer into another file, a fully-substituted type. Avoid this. Resolved cross-file data captured at build time goes stale: the other file changes, the identity it referred to no longer exists, and anything comparing against it silently rejects genuine matches. The most visible victim is [Find Usages](Searching.md), whose pass 2 confirms a candidate by comparing declared elements - a comparison a stale cached identity will fail.

The idiomatic pattern is **cache names, resolve live**: persist only lightweight, local, textual facts - short names, offsets, kinds, scope paths, hashes - and resolve live the cross-file relationships on demand from those facts when they are actually needed. Single-file analysis stays fast, the persisted data cannot encode a stale cross-file identity, and resolution always reflects the current state of the other files.

C++ follows this exactly. `CppGlobalSymbolCache` persists a set of word *hashes* per file (plus per-file symbol tables of names and offsets), not resolved cross-file links:

```csharp
private readonly OptimizedPersistentSortedMap<CppFileLocation, IntSet> myWordsMap;
// ...
myWordsMap = solutionCaches.Db.GetMap("CppWords", CppFileLocationUnsafeMarshaller.Instance, IntSet.Marshaller)
  .ToOptimized(lifetime);
```

The stored `int` values are hashes of the words, not resolved elements. During search, C++ resolves live each candidate reference and compares declared elements (see [Reference search](ReferenceSearch.md)) rather than trusting anything resolved at cache-build time. This is what keeps Find Usages correct across edits and reloads - and, together with a correctly bumped `Version`, across restarts.
