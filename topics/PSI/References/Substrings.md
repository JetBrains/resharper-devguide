[//]: # (title: References within elements - substrings)

A reference doesn't need to be applied to the complete range of an element - it doesn't have to be the whole string in a string literal. Multiple references can be applied to different parts of a single element.

For example, a file path reference provider will create multiple references for substrings within the string, one for each file path segment. A reference which wishes to do this should implement the `IReferenceWithinElement<T>` interface, where `T` is the owning PSI tree node.

```csharp
public interface IReferenceWithinElement : IReference
{
  ITreeNode Token { get; }
  TreeTextRange RangeWithin { get; }
}

public interface IReferenceWithinElement<T> : IReferenceWithinElement
  where T : class, ITreeNode
{
  ElementRange<T> ElementRange { get; set; }
}
```

The constructor for `ElementRange` takes in an instance of `T` and the `TreeTextRange` that is the range within the node that should be used for the reference. The range should be relative to the length of the owner node's short name.

Instead of implementing the `IReferenceWithinElement<T>` interface directly, you can derive from `ReferenceWithinElementBase<TOwner, TToken>`, which provides most of the implementation for you. This class derives from `TreeReferenceBase` but implements things a little differently.

 >  In the type signature for `ReferenceWithinElementBase<TOwner, TToken>`, `TOwner` is the owner of the reference, and `TToken` is the node within the owner's sub-tree that will be used for the reference region. For example, `TOwner` can be `ICSharpLiteralExpression` and `TToken` would be `ITokenNode`, and set to a value of `ICSharpLiteralExpression.Literal`.
 >
 {type="note"}

The constructor takes in the two element nodes, and the relative `TreeTextRange` of the range within the `TToken` tree node parameter. The name of the reference, as returned by `GetText` is the substring of the `TToken` node defined by this range. The range of the reference comes from the relative tree range of the substring.

`ReferenceWithinElementBase` provides an implementation of the abstract `ResolveWithoutCache`. It defers to the abstract `GetReferenceSymbolTable`. However, after retrieving the symbol table, it applies filters returned from `GetSymbolFilters`. This allows our implementation of `GetReferenceSymbolTable` to be very straight forward - we can return the symbol table for a particular class, for example, and then use the filters to reduce it to appropriate candidates.

The core implementation of `GetSymbolFilters` is held in `GetCompletionFilters` which returns an empty list by default. To add functionality, return an array of filters, for example:

```csharp
return new ISymbolFilter[]
{
  new DeclaredElementTypeFilter(ResolveErrorType.NOT_RESOLVED, CLRDeclaredElementType.ENUM_MEMBER),
  IsPublicFilter.INSTANCE
};
```

These will create a filter that finds any enums, and returns `ResolveErrorType.NOT_RESOLVED` if it can't find anything. If it finds an enum, it then goes through the `IsPublicFilter` instance, which checks that the declared element is public. If it can't find a match, it returns `ResolveErrorType.ACCESS_RIGHTS`. There are lots of implementations of `ISymbolFilter` which can be sequenced together to filter down to the appropriate candidates.

The only other method that requires implementation (unless you wish to customise the class further) is `BindToInternal`. The `BindTo` methods are implemented, and call into `BindToInternal`. This method can use the `ReferenceWithinElementUtil.SetText` method to set the text in the substring of the node, by creating a new token, with a passed in token factory `Func`, using the new text of the node. It also fixes up any other references that are attached to the original node.

However, there is additional housekeeping required when the change is happening in a file that can contain multiple PSI trees (e.g. HTML files can contain HTML, JS and CSS trees). In this case you should use either the `HtmlReferenceWithTokenUtil.SetText` or `JavaScriptReferenceWithTokenUtil.SetText` extension methods to affect the change, as they propogate the change to the other PSI trees. Both implementations are the same, and it would be a reasonable choice to always call these methods instead of `ReferenceWithinElementUtil` directly.

 >  The change is propogated to the other PSI trees by virtue of adding a "cookie" to the current PSI transaction. The `CreateCustomCookie` extension method is used to attach an object to the transaction's list of "cookie" objects. When making the change, the code asks the PSI transaction if the cookie is set, and if so, propogates the changes to the other PSI files
