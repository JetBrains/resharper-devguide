[//]: # (title: Word index)

The word index is what makes pass 1 of the [two-pass search model](Searching.md) fast. It is an index of the words used in each file, so the finder can ask "which files even mention this name?" and skip everything else without parsing it. The good news for a language author is that participation is automatic - there is nothing to register.

## `IWordIndex`

`IWordIndex` is one of the standard PSI caches, retrieved from `IPsiServices.WordIndex`. It answers file-narrowing queries such as "which files contain all of these subwords?" or "which files contain any of these subwords?".

The [finder](Searching.md) uses these queries together with the [search words](ReferenceSearch.md) a language returns from `GetAllPossibleWordsInFile`: only files that the index says contain those words are handed to the reference searcher for pass 2.

## Participation is automatic

There is **no** per-language word-index provider to implement or register - no word scanner, no `IWordIndexLanguageProvider`. The index is a single solution-wide component that indexes the raw text of every applicable PSI file, and it is [persisted to disk](Caching.md) like any other cache.

Applicability is based only on generic file properties, so any language whose files build a PSI, provide a code model, and take part in the caches is indexed for free. The "words" are extracted generically from the file text by splitting on identifier characters - the first character must be a letter, the following characters a letter, digit or underscore.

## What a language actually provides

Since indexing is automatic, a language's only responsibility for pass 1 is to supply the *query* words - the names to look for - through [`GetAllPossibleWordsInFile`](ReferenceSearch.md). Getting those query words right is the whole game for pass 1: the index only stores plain identifier subwords, so a query word that never appears verbatim in the text matches no files and the usage is silently missed. See [Reference search](ReferenceSearch.md) for the rules on which word-forms to return - for example why C++ returns a destructor name both with and without its leading tilde.

## See also

* [Reference search](ReferenceSearch.md) - where the query words come from (`GetAllPossibleWordsInFile`) and how pass 2 confirms the candidates.
* [Search domain](SearchDomain.md) - the other half of pass 1, bounding which files are considered at all.
* [Caching](Caching.md) - the word index is a persisted cache like any other.
