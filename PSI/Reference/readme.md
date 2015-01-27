# PSI Reference

> **NOTE** This is not intended to be part of the compiled documentation!

The files in these folders are generated semi-automatically. The PsiDocs2 program (not included in the repo yet, speak to Matt Ellis) will create and try to update the files. The process is:

* The program uses Mono.Cecil to load the assembly containing the PSI implementation
* It finds types:
    * Tree nodes - all interfaces deriving from `ITreeNode`.
    * Navigators - all classes with a name that ends `Navigator`.
    * Utility classes and extension methods - all classes with a name ending `Util`, `Extension` or `Extensions`.
    * Element factories - all types where the type name contains `ElementFactory` or `NodeFactory`, but doesn't end in `Extension` or `Extensions`.
* It then creates or updates a type index file for each category of type.
* Finally, any types listed in any file in the output directory are updated.

## Generating type index files

The program will first look to see if any files exist for the given category. For example, when creating the type index for tree nodes, the program will check to see if any `TreeNodes*.md` files already exist. This will pick up the type index if it's concatenated into a single file, or if it's split across multiple files.

If the files do not exist, they are created. The types are grouped alphabetically (if all types are interfaces, the leading 'I' is stripped). There is some logic to try and balance the number of files with the size of the files, but by default, the types are grouped in files such as:

* `TreeNodesA-D.md`
* `TreeNodesE-H.md`
* `TreeNodesI-L.md`
* `TreeNodesM-P.md`
* `TreeNodesQ-T.md`
* `TreeNodesU-Z.md`

The file is initially created as "auto-indexed", which means that when it's updated with a new version of the PSI, any missing types are automatically added in the appropriate place. This is done by adding the following HTML comment to the top of the file:

```
<!-- Index A - D (auto-generated. Remove this line if manually adding/removing entries) -->
```

The file then adds the `<!-- toc -->` table of contents macro used by the [gitbook-plugin-toc](https://github.com/whzhyh/gitbook-plugin-toc) GitBook plugin.

Each letter in the index then gets a new heading, and each type definition is added in between the special HTML comments `<!-- Begin {TypeName} -->` and `<!-- End {TypeName} -->`. These comments are used when updating the content.

## Updating type index files

If the index files already exist, they need to be updated, in order to ensure that they contain only the required types, and that the type definition itself is up to date. Each file is processed, looking for the auto-index comment (see above). If this comment is found, the letters after the word "Index" are used to indicate that this file contains types beginning with those letters (again, interaces have the leading 'I' removed, only if *all* types are interfaces, e.g. tree nodes).

If the auto-index comment is missing, the app will create a new file called `{Type}-missing.md` (e.g. `TreeNodes-missing.md`). This file will contain a list of types that are missing from the other files. They will be correctly formatted, and should be manually copy-and-pasted to the appropriate place.

Each file is now processed. Auto-indexed files will automatically have new types added, and any types that no longer exist are removed, even for non-auto-indexed files. When adding a type, a new section is added, with header, HTML comments and the type definition. When removing a type, the whole section is removed, including header, HTML comments, type definitions and *any* other content up to the next section. Please make sure to review changes before committing!

## Updating content

Finally, the program will update the content in all `*.md` files in the output directory, not just the index files created. This includes hand-crafted files such as `Overview.md`. It will process all `*.md` files in the output directory and look for specially formatted HTML comments, that gives the name of the type that should be output between the comments.

For example, the content:

```
<!-- Begin ICSharpTreeNode -->
<!-- End ICSharpTreeNode -->
```

will be replaced with:

```
<!-- Begin ICSharpTreeNode -->
public interface ICSharpTreeNode : ITreeNode
{
  // Snipped for brevity
}
<!-- End ICSharpTreeNode -->
```

As long as the type is recognised from the list of types processed from the PSI assembly, it will be output. If the type is not recognised, an error is written to `stdout` and should be investigated.

## Adding a new language

To add a new language:

* Add a new folder for the language
* Run the PsiDocs2 app on the PSI assembly and output to this language folder
* Do a manual clean up
    * Combine index files that are too short, or don't have enough entries. E.g. the XML PSI doesn't have many nodes or navigators, so the index files have been combined into single files - `Navigators.md` and `TreeNodes.md`. Make sure to update the auto-index comment to reflect the starting letters of the entries in that file.
    * Delete any empty files. If there are no utility classes, or element factories, delete the appropriate files (`Utils.md`, `ElementFactories.md`, etc.)
* Annotate the index files. Manually edit the index files and add brief comments for each class. Make sure to add the text after the end HTML comment `<!-- End ICSharpTreeNode -->`
* Add the `Overview.md` and `Manipulation.md` files to provide article style docs to provide an overview of the AST and how to manipulate the tree. The other files are intended to be reference files. See existing files for examples.
* Add the files manually to `SUMMARY.md`

## Updating a language

When a new version of ReSharper is released, simply run PsiDocs2 and replace the existing content. Review the changes in a `git` tool. The changes are likely to be:

* Changes to the type. Make sure the text content for that interface is still applicable.
* Removed types. Update the index file to remove the section for that type, and update any references in non-index files, e.g. `Overview.md`
* Added types. If the index files are auto-indexed (that is, the auto-index comment at the top of the file is still in place), the type will be added to the file, in the correct place alphabetically. If the index files are not auto-indexed (the auto-index comment has been removed), the type will be saved to a `-missing.md` file, where it can be manually copy-and-pasted to the appropriate file. Make sure to add a brief text section describing the type.

## Editing tips

* When manually creating a file such as `Overview.md`, types can be referenced simply by adding the HTML comments and running the PsiDocs2 app. This will update the file to contain the type definition.
* The index files can be split across any number of files. If the app does not create an optimal balance of files vs number of entries per file, then the files can be manually edited. As long as the auto-index comment exists, and reflects the letters of the entries in that file, the app will update the new index files appropriately.
* You can easily create index files ordered as you see fit by editing the auto-index comment to include the start and end index character for that file. For example, setting it to `<!-- auto-index A - Z` will include all items from A - Z.

## Why disable auto-index?

The auto-index scheme was added for the XML PSI, to allow for two indexes, one for the XML AST, and the other for the DTD AST. Since the types overlap in names, they can't be split alphabetically, so the indexes are maintained (semi-)manually. The auto-index comment is deleted, meaning the PsiDocs2 app will output any missing types to a separate file, where they can be manually moved to the appropriate file.
