# Type systems

The `IDeclaredElement` interfaces provide a semantic view of something that has a declaration, but that doesn't necessarily allow for modelling *usages* of the element. Specifically, if we take a look at CLR types, a declared element is a declaration of a type, such as a class, but it can't represent all usages of the type, such as in an array, pointer or closed generic type. The `ITypeElement` interface can provide a semantic view of the declaration of `Foo` or `Bar<T>`, but it can't represent `Foo[]` or `Bar<string>`.

Declared elements need to be able to model these usage scenarios to provide information about base classes, method signatures, etc. In order to do so, the derived declared elements (such as `ITypeElement`) use an additional interface hierarchy to represent this "type system" information. This hierarchy is language specific, like the derived declared elements that use them. For CLR types, it's the `IType` hierarchy, for JavaScript, it's `IJavaScriptType`.

This `IType` is additional information rather than a replacement for the declared element's semantic view. While the `IType` can return a symbol table of all type members, it doesn't provide accessors in the same way that `ITypeElement` does. However, it is possible to get back to the declared element. If the type implements `IDeclaredType`, the `GetTypeElement` method will return the `ITypeElement`, which gives the full semantic view of the type.

Arrays and pointers are both defined in terms of another instance of `IType`, allowing for recursively declaring types, such as multi-dimensional arrays, or arrays of generic types. The derived interfaces, such as a `IArrayType` allow getting the element type (such as `int` in `int[]`), and in the case of arrays, the array's rank.

Generic types are internally represented by a declared element and an instance of `ISubstitution`, which describes how the generic type parameters are substituted. (Non generic types are represented in the same manner, but use the Null Object Pattern and use an instance of `EmptySubstitution`)

It's also possible to get the underlying declared type via the `IType.GetScalarType` method, which returns the element type for arrays, and will downcast to `IDeclaredType` for other types.

Types are retrieved from a declared element - such as a base type, or member type signature. Alternatively, types can be created using the `TypeFactory.CreateType` methods, passing in a fully qualified type name and any type parameters, or an `IDeclaredElement` and an instance of `ISubstitution`. Some frequently used types are also available from the `PredefinedType` class.
