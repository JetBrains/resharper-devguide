[//]: # (title: Search domain)

A *search domain* bounds the set of files a search looks in. It is the second half of pass 1 (the [word index](WordIndex.md) is the first): the finder only ever hands the [reference searcher](ReferenceSearch.md) files that are both inside the domain and survive the word filter. Choosing the domain well is what makes search both fast (a small domain) and correct (a domain that actually contains the usages).

## `ISearchDomain`

A domain is an immutable set of files with set-algebra operations (`Union`, `Intersect`, `Contains`), defined in `JetBrains.ReSharper.Psi.Search`. A language plugin does not implement `ISearchDomain` itself - it creates one through `SearchDomainFactory` (reachable from `IFinder.SearchDomainFactory` or `IPsiServices.SearchDomainFactory`), whose `CreateSearchDomain` overloads build a domain from a solution, projects, PSI modules, source files, or tree nodes. Because domains support `Union`/`Intersect` and compare by value, the finder can merge the domains contributed by different factories into one.

## `GetDeclaredElementSearchDomain`

`IDomainSpecificSearcherFactory.GetDeclaredElementSearchDomain` returns the domain for a specific target element. The finder unions the domains from every compatible factory, so returning the empty domain (`EmptySearchDomain.Instance`, the base-class default) simply means "this factory contributes no files".

The correctness pitfall here is the single most common reason Find Usages returns nothing for a custom language in a real solution:

> Do **not** build the domain purely from `solution.GetAllProjects()` (or the project files of one file type) if your language's files are not plain members of the project model. Generated files, files pulled in from packages, external or include-only sources, and files owned by a language that layers on top of another project system are frequently *not* project-model members. A domain built from `GetAllProjects()` is then nearly empty, the finder is handed almost no candidate files, and usages that plainly exist are never found - while a handful that happen to live in a real project still work, giving the confusing "some usages work, most don't" symptom.
>
{type="warning"}

The robust approach is to build the domain from the language's own view of its files - its symbol/index caches and its PSI modules - rather than from the project model. **PSI modules** are the right abstraction: every file that has PSI belongs to a PSI module, including files that are not project-model members.

## C++ example

C++ never builds its domain from `GetAllProjects()`. `CppSearcherFactory.GetDeclaredElementSearchDomain` delegates to a visitor over the C++ declared element:

```csharp
public override ISearchDomain GetDeclaredElementSearchDomain(IDeclaredElement declaredElement)
{
  if (declaredElement is CppPathDeclaredElement pathElement)
  {
    if (ProjectFilesToRename(pathElement).Any())
      return mySearchDomainFactory.CreateSearchDomain(declaredElement.GetSolution(), false);
  }
  else if (declaredElement.IsClrElement())
    return CppDeclaredElementUtil.GetSearchDomain(CppDeclaredElementUtil.TryCreateClrLinkageEntityElement(declaredElement));

  return CppDeclaredElementUtil.GetSearchDomain(declaredElement);
}
```

For the broad, solution-wide case the domain is built from **PSI modules**, not projects - so files that are not plain project members still participate (`CppDeclaredElementUtil.GetSolutionSearchDomain`):

```csharp
private ISearchDomain GetSolutionSearchDomain(ICppDeclaredElement e)
{
  var services = e.GetPsiServices();
  var modules = services.Modules.GetModules();
  var factory = e.GetPsiServices().SearchDomainFactory;
  return factory.CreateSearchDomain(modules.Where(IsCppModule).Where(module => module.SourceFiles.Any()));
  // ...
}
```

For symbols whose usages can only appear in files that include the declaration (an anonymous-namespace member, a `static` global), C++ narrows the domain further using its own **include graph** from `CppGlobalSymbolCache`, so the search touches only the files that transitively include the target's file (`CppDeclaredElementUtil.GetSearchDomainForIncluders`):

```csharp
private ISearchDomain GetSearchDomainForIncluders(ICppDeclaredElement e)
{
  var solution = e.GetSolution();
  var symbolCache = solution.GetComponent<CppGlobalSymbolCache>();
  return e.GetPsiServices().SearchDomainFactory.CreateSearchDomain(
    e.GetSymbols()
      .SelectMany(s => symbolCache.IncludesGraphCache.CollectAllIncludersAndLocalIncluders(s.ContainingFile))
      .Distinct()
      .SelectMany(f => f.GetAllSourceFiles(solution, symbolCache.CppModule).ToEnumerable())
      .Distinct());
}
```

The two ideas to take away are:

* **Source the domain from the language's own index / PSI modules**, so files outside the project model are included.
* **Narrow the domain when you can** (here, to only the includers of the target), so the search stays fast. The word index will narrow within the domain too, but a tighter domain avoids even looking at unrelated files.

## See also

* [Word index](WordIndex.md) - the other half of pass 1, filtering the domain by text.
* [Reference search](ReferenceSearch.md) - how the domain is consumed by the reference searcher.
* [Searching](Searching.md) - the overall two-pass model.
