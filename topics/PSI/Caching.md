[//]: # (title: Caches)

ReSharper makes heavy use of caching in order to store all the information required to provide a semantic view of a codebase. The caches are calculated on solution load, stored to disk, and reloaded when the solution is subsequently loaded. Any time a file changes, the cache is notified, and can rescan the file and update its details.

The PSI implements the infrastructure for building and maintaining a cache, and provides a number of different caches, such as `ISymbolCache` - a symbol cache for looking up CLR `ITypeElement` declared elements, and `IWordIndex`, which is an index of words and the files they are used in (one use of this is reverse lookup of references).

The cache infrastructure is extensible, and other parts of the architecture create their own caches on top of this infrastructure. For example, the To Do Explorer maintains a cache that initially scans all files for to-do items, and keeps the list of items up to date as files change.

Caches are very useful for cross-file analysis, amongst other things. Normal analysis is performed in the context of a single file, and is called very frequently as the file is edited. It should therefore be as fast as possible, and not try to scan other files.

For example, imagine a scenario where an analyser is trying to ensure uniqueness of assembly level attributes. When a file is changed, the analyser can look in that file for the required attribute, but it would be very bad for performance if it would also try to scan all other files in the project for the same attribute. Instead, a cache can be implemented to initially scan for all attribute instances, and used by the analyser to look up potential duplicates.

The PSI caches can be retrieved by using `IPsiServices`, which provides direct access to `ISymbolCache` via `IPsiServices.Symbols`, or `IWordIndex` via `IPsiServices.WordIndex`. Other caches can be injected directly into a class constructor. 

A similar service is the `IPersistentIndexManager`, which is an extensible, persistent cache that is not related to file contents. For example, the unit test runner uses a persistent index to store results for each test. A persistent index is a disk backed key-value store (implemented on [LevelDB](http://en.wikipedia.org/wiki/LevelDB) although this is an implementation detail) that automatically manages what data is in memory or flushed to disk. The interface is a simple dictionary-like lookup.

Finally, the PSI maintains a set of in-memory caches. These are lightweight, transient caches that aren't persisted and are invalidated when any abstract syntax tree changes. They are intended to temporarily cache the result of operations that might occur again. Generally speaking, these caches are not intended for consumption from extensions, as they are used by other parts of the code. However, the cache can be retrieved either by injecting the component into a class constructor or by using `IPsiServices.Caches.GetPsiCache<T>` where `T` is the required cache, which implements `IPsiCache`, such as `InternalsVisibleToCache`.
