[//]: # (title: Reference search)

To take part in Find Usages and the other search-based features, a language implements a *domain specific searcher factory*. This is the single extension point the [finder](Searching.md) uses to ask a language for the language-specific pieces of a search: the words that feed pass 1, the [search domain](SearchDomain.md), and the reference searcher that performs pass 2.

## `IDomainSpecificSearcherFactory`

The factory is defined in `JetBrains.ReSharper.Psi.ExtensionsAPI`:

```csharp
public interface IDomainSpecificSearcherFactory
{
  bool IsCompatibleWithLanguage(PsiLanguageType languageType);

  // words that must occur in a file for it to contain a reference to `element` (pass 1)
  IEnumerable<string> GetAllPossibleWordsInFile(IDeclaredElement element);

  // the reference searcher that confirms usages in each candidate file (pass 2)
  IDomainSpecificSearcher CreateReferenceSearcher(IDeclaredElementsSet elements,
    ReferenceSearcherParameters referenceSearcherParameters);

  // which files can possibly contain usages
  ISearchDomain GetDeclaredElementSearchDomain(IDeclaredElement declaredElement);

  // ... plus text-occurrence, related-element, derived-request and navigation hooks
}
```

The finder collects every registered factory in the solution (`IFinder.DomainSpecificSearcherFactories`) and, for a given search, uses each factory whose `IsCompatibleWithLanguage` returns `true`. Search domains and search words from different factories are merged, so more than one language or technology can contribute to a single search.

Rather than implementing the interface directly, derive from `DomainSpecificSearcherFactoryBase`: all of its members default to an empty/`null` result (`GetAllPossibleWordsInFile` returns an empty list, `CreateReferenceSearcher` returns `null`, `GetDeclaredElementSearchDomain` returns `EmptySearchDomain.Instance`), so a language overrides only what it needs.

Register the factory as a PSI component so the finder can discover it. From `CppSearcherFactory.cs`:

```csharp
[PsiComponent(Instantiation.DemandAnyThreadSafe)]
internal class CppSearcherFactory : DomainSpecificSearcherFactoryBase
{
  public override bool IsCompatibleWithLanguage(PsiLanguageType languageType)
  {
    return languageType.Is<CppLanguage>() || languageType.Is<CppDoxygenLanguage>();
  }

  // ...
}
```

## Search words feed pass 1

`GetAllPossibleWordsInFile` returns the words that *must* appear textually in a file for it to possibly contain a reference to the element. The finder passes these words to the [word index](WordIndex.md), which discards any candidate file whose text does not contain them - this is pass 1. Usually the answer is simply the element's short name, but a language can return several words, or apply its own naming rules.

`CppSearcherFactory.GetAllPossibleWordsInFile` illustrates the pattern (from `CppSearcherFactory.cs`):

```csharp
public override IEnumerable<string> GetAllPossibleWordsInFile(IDeclaredElement element)
{
  element = CppDeclaredElementUtil.GetDeclaredElementForSearch(element);
  if (element is ICppDeclaredElement cppElement)
    return GetDeclaredElementNames(cppElement);
  if (element is IPathDeclaredElement pathElement)
    return new [] { pathElement.Path.Name.ToLowerInvariant(),
                    pathElement.Path.NameWithoutExtension.ToLowerInvariant() };
  return EmptyList<string>.InstanceList;
}
```

Note the extra care C++ takes for destructors, where the word index does not store the leading tilde, so both forms are returned (`CppSearcherFactory.GetDeclaredElementNames`):

```csharp
else if (namePart.IsDestructorTag())
{
  var shortName = namePart.GetShortName();
  // Name without tilde is necessary because word index does not contain destructor names
  return new[] { shortName, shortName[1..] };
}
```

> This method is the most common cause of "Find Usages finds nothing" for a new language. If a returned word does not match how the name is written at the usage site (case, qualification, an operator or accessor spelling, a decorating character), pass 1 drops every file that actually contains the usage, and the usage is silently missed - no error is shown, because resolution itself is fine. Return `null` or `string.Empty` in the enumeration to switch word filtering off and fall back to scanning every file in the domain (correct, but slow).
>
{type="warning"}

## The reference searcher performs pass 2

`CreateReferenceSearcher` builds an `IDomainSpecificSearcher` for the set of target elements. The searcher is handed each candidate file that survived pass 1, and its job is to *confirm* the usages. From `CppSearcherFactory.cs`:

```csharp
public override IDomainSpecificSearcher CreateReferenceSearcher(IDeclaredElementsSet elements,
  ReferenceSearcherParameters referenceSearcherParameters)
{
  var searchTargets = new CppSearchTargets(elements);
  if (searchTargets.ElementsToSearch.IsEmpty())
    return null;
  return new CppReferenceSearcher(myLogger, this, searchTargets,
    visitReferencesFromMacroArgument: true, visitDoxygenReferences: true);
}
```

The searcher asks the factory for the same search words to build a fast in-file text pre-filter, then walks the PSI of the candidate file. The actual confirmation happens per reference in the source-file processor. This is the heart of pass 2: for every candidate reference, *resolve it live* and accept it only when the resolved declared element is one of the search targets. From `CppReferenceSearchSourceFileProcessor.ProcessReference`:

```csharp
protected override FindExecution ProcessReference(IReference reference)
{
  IEnumerable<IDeclaredElement> resolved = CppReferenceClassifier.Resolve(reference, out var info).AsIList();

  foreach (var resolvedElement in resolved)
  {
    var element = CppDeclaredElementUtil.GetDeclaredElementForSearch(resolvedElement);
    if (Elements.Contains(element))
    {
      // resolved back to a search target - report it as a usage
      return myResultConsumer.Accept(new FindResultReference(reference, element));
    }
  }

  return FindExecution.Continue;
}
```

Two things make this step correct:

* **Resolve live, not a name match.** A textual name match is only a hint; the searcher resolves the reference and compares *declared elements*. This is why two members with the same short name in different scopes do not become each other's usages.
* **Stable declared-element identity.** `Elements.Contains(element)` relies on the resolved element comparing equal (`Equals`) to the target. If a language's declared element does not have a stable identity across resolution - for example if it keys identity on the *referencing* file, or on data restored from a stale [cache](Caching.md) - the comparison fails and a genuine usage is rejected. See [Caching](Caching.md) for how to keep persisted data from breaking this comparison.

## See also

* [Search domain](SearchDomain.md) - how `GetDeclaredElementSearchDomain` chooses the candidate files.
* [Word index](WordIndex.md) - what the search words are matched against in pass 1.
* [Caching](Caching.md) - keeping cross-file resolution (and therefore pass 2) correct.
* [Navigation](Navigation.md) - how confirmed usages are presented as occurrences.
