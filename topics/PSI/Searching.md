[//]: # (title: Searching)

Many of ReSharper's most visible features - Find Usages, Ctrl+Click navigation, Rename and other refactorings - are all built on top of a single mechanism: searching for the [references](References.md) to a declared element. This section documents the *search engine* that powers these features, and what a custom language plugin has to provide so that its usages are actually found.

> This is the engine side of the story. The presentation side - occurrences, occurrence kinds, the Find Results window - is covered in [Navigation](Navigation.md).
>
{type="note"}

## The two-pass model

Searching for the usages of a declared element over a whole solution one reference at a time would be far too slow - it would mean re-parsing and re-resolving every reference in every file. Instead, the finder uses a two-pass model:

1. **Pass 1 - narrow the candidate files.** The finder asks the [word index](WordIndex.md) which files textually contain the short name(s) of the target element. A file that does not even mention the name by text cannot contain a reference to the element, so it is skipped without ever building its PSI. This is what makes search fast: only a handful of files usually survive.
2. **Pass 2 - confirm each candidate.** For every surviving file, the finder walks the PSI, and for each reference whose name matches, it *resolves the reference live* and compares the resolved declared element against the search target. Only references that resolve back to the target are reported as usages.

The important consequence for a language plugin is that both passes have to agree. Pass 1 depends on the searcher contributing the right *search words*; pass 2 depends on the language's references resolving to a declared element with a *stable identity*. If either is wrong, usages are silently missed even though nothing is highlighted as an error.

```text
                         target IDeclaredElement
                                   |
                                   v
                    IDomainSpecificSearcherFactory
                       |                        |
   GetAllPossibleWordsInFile        GetDeclaredElementSearchDomain
     (short names)                    (candidate files)
        |                                     |
        v                                     v
   IWordIndex  ----------.        .---- ISearchDomain
                          \      /
                           v    v
             Pass 1: narrow the candidate files
                           |
                           v
             IReferenceSearcher over each candidate file
                           |
       Pass 2: resolve each reference live and compare declared element
                           |            ^
                           v            | resolution reads
                confirmed usages        | persisted caches
                (occurrences)           | (see Caching)
```

## What a plugin contributes

Most of this engine is not the plugin's concern. The entry point is `IFinder` (from `IPsiServices.Finder`), which exposes the search operations (`FindReferences`, `FindInheritors`, `FindImplementingMembers`, `FindTextOccurrences`, and so on); a caller starts a search by handing it a target `IDeclaredElement`, an [`ISearchDomain`](SearchDomain.md), and an `IFindResultConsumer<TResult>`. But the finder is entirely part of the platform - **a language plugin never implements it, and rarely even calls it directly** (the search-based features do that). It is language agnostic and delegates every language-specific decision to the `IDomainSpecificSearcherFactory` instances it collects from the solution. So the one thing a plugin actually contributes to search is that factory - the rest of this section is about getting it right.

For each search, the finder walks its `DomainSpecificSearcherFactories` and asks every compatible factory for the language-specific parts of the search:

* [`IDomainSpecificSearcherFactory`](ReferenceSearch.md) is the language's plug-in point into the engine. It declares which languages it handles (`IsCompatibleWithLanguage`), which search words feed pass 1 (`GetAllPossibleWordsInFile`), which files to look in (`GetDeclaredElementSearchDomain`), and it creates the actual reference searcher (`CreateReferenceSearcher`).
* The [word index](WordIndex.md) uses the search words to narrow the candidate files (pass 1). Participation in the word index is automatic for any language that builds a PSI - there is nothing to register.
* The [search domain](SearchDomain.md) bounds and further narrows the file set. This is the place where a language decides *which* files can possibly contain usages - including files that are not plain members of the project model.
* The reference searcher processes each candidate file and confirms usages by resolving references live and comparing declared elements (pass 2).

Resolution in pass 2 is where cross-file [caches](Caching.md) come in: to resolve a reference in a candidate file, the language usually needs cross-file symbol information. How those caches are built, versioned, and kept from going stale directly determines whether pass 2 accepts or rejects a candidate - see [Caching](Caching.md) for the "cache names, resolve live" guidance that keeps this correct.

## In this section

* [Reference search](ReferenceSearch.md) - the searcher factory and reference searcher, search words, and pass-2 confirmation.
* [Search domain](SearchDomain.md) - how the candidate file set is chosen, including files outside the project model.
* [Word index](WordIndex.md) - the word index and how pass 1 narrows candidate files for free.
