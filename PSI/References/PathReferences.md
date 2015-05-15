---
---

# Path references

ReSharper supports path references, which are references that target file system paths. They implement `IPathReference` or `IFileReference`, which both inherit from `IReference` and typically apply to a substring within a string literal. Hitting Ctrl+Click on a path reference will navigate to that file or folder, and renaming a path (as long as it's part of the Visual Studio project) will be reflected in the string associated with the reference.

Path references are also "qualified" references. That is, the reference has a qualifier, which is the previous part of the path. The value of these qualifiers are available via the `IQualifiableReferenceBase` and `IReferenceWithQualifier` interfaces, which inherit from `IReference` and are implemented by the path reference. The qualifiers themselves implement `IQualifier`, and are used during when the reference is resolved. (Note that qualified references are not necessarily file path references. For example, ASP.NET support has data binding references where each segment of the reference uses the previous segment's reference as a qualifier.)

## Creating path references

There are several helper classes for creating file and folder references, depending on the language of the underlying document. The `PathReferenceUtil.CreatePathReferences` static method will parse a path string and generate references for the folder and file segments.

```csharp
public static IPathReference[] CreatePathReferences<TOwner,TToken>(
  TOwner owner, TToken token, IQualifier baseQualifier,
  Func<TOwner, IQualifier, TToken, TreeTextRange, IPathReference> createFolderReferenceDelegate,
  Func<TOwner, IQualifier, TToken, TreeTextRange, IPathReference> createFileReferenceDelegate,
  Func<TToken, string> getStringValueDelegate,
  Func<TToken, int> getValueStartOffset
)
  where TOwner : ITreeNode
  where TToken : class, ITreeNode
{
  // ...
}
```

The parameters are as follows:

* **`owner`** and **`token`** are two `ITreeNode` instances that represent the owner of the references, and the token that contains the path. These are not necessarily the same tree node. For example, an HTML attribute would own the references, but it's the string literal token part of the attribute that is used to get the path string.
* **`baseQualifier`** represents the part of the path that this current path is relative to. It might be a root part of the path, or an implied reference. (e.g. `~` in a web site implies the root of the project, and the string path is relative to the root of the project)
* **`createFolderReferenceDelegate`** and **`createFileReferenceDelegate`** are two functions that are called to create a folder and file reference, respectively. The parameters to these functions are:
    * the `ITreeNode` that will own the references
    * the qualifier of this path segment
    * the token that provides the string
    * the `TreeTextRange` of the path segment inside the token, relative to the start of the PSI tree, not the token.

  The functions should create an instance of `IPathReference`, or return null. If the reference also implements `IQualifier`, it is passed to the creation function as the qualifier for the next path segment.
* **`getStringValueDelegate`** and **`getValueStartOffset`** are used to get the string value of the whole path to parse from the token, and the start offset within the token of the path.

The `CreatePathReferences` method will first try to parse the path as a URI. If it is a valid, absolute URI, the path references are retrieved by iterating through all `IUriPathResolver` solution components until one returns a non-null value. The list of components are in an arbitrary order, and ReSharper only provides a single implementation - `SdkReferenceContentFilePathResolver`, which looks for `ms-appx://` Windows Store App SDK references. It will create a reference for the first path segment, then call back into `PathReferenceUtil.CreatePathReferences` method to parse the rest of the path, passing the first path segment's reference as the base qualifier. If no `IUriPathResolver` components handle the URI, no references are created for it.

Assuming the path isn't a valid, absolute URI, it is then split into path segments based on the directory char separators `\` and `/`. All segments apart from the last are passed to `createFolderReferenceDelegate`. If the delegate creates a reference, it is checked for `IQualifier` and used as the qualifier for the next path segment. The delegate can also return null, in which case, no reference is created for this path segment. The first path segment receives the `baseQualifier` parameter, unless it matches the `~` string, in which case the qualifier is set to `null`.

The last path segment is passed to `createFileReferenceDelegate`, unless it matches `~`, `.` or `..` in which case it's assumed to be a folder, and is passed to `createFolderReferenceDelegate`.

## HTML and HTML-like languages

The `HtmlPathReferenceUtil` provides wrapper methods for `PathReferenceUtil.CreatePathReferences` for HTML and HTML-like languages (ASPX, Razor, etc.). The `CreatePathReferences` method is a simple pass through, but the `CreatePathAndIdReferences` method will use `PathReferenceUtil.CreatePathReferences` to create the file and folder references, and create an additional reference for the #-fragment of the URL, pointing to a HTML element based on its ID.

The HTML languages also provide their own implementations of the reference classes, with individual knowledge of the language they are supporting. They all derive from `PathReferenceBase` or `HtmlFileReference` or `HtmlFolderReference`.

If you're not working with HTML or HTML-like languages, and `PathReferenceBase` isn't appropriate to derive from, you can derive from `QualifiableReferenceWithinElement` to provide a base implementation of `IReferenceWithQualifier`.

