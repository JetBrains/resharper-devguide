[//]: # (title: Type System)



Any kind of manipulation of object-oriented structures necessarily involve working with _types_ such as `int`, `string` or, say, `MyType<U,V>`. Predictably, ReSharper has a number of data structures that allow plugin writers to identify and introspect types, whether these types are available as code or not.

 >  use the _Explore Type Hierarchy_ (*Ctrl+E,H*) command to explore the hiearchies of the various `IXxxType` interfaces.
 >
 {type="tip"}

## IExpressionType

![Type hierarchy for IExpressionType](iexpressiontype_hierarchy.png)

At the very top of the ‘hierarchy of types’, we have the `IExpressionType` interface. This is a wide-reaching interface that covers any type that an expression may be and also happens to cover the principal ‘concrete’ interface `IType`.

Apart from `IType`, the interfaces and classes directly implementing `IExpressionType` are typically language-specific. Examples of such expression types include C#‘s `EventType` or VB’s `IVBNothingType`.

The `IExpressionType` defines a fundamental contact which all expression types adhere to. Thus, it makes sense to describe some of its members.

* `IsResolved` - this property tells us whether this type has actually been resolved to something. For example, if the expression type is `StringBuilder`, ReSharper will resolve it to `System.Text.StringBuilder`. However, if the expression type is `Foo` and this class does not exist yet, `IsResolved` will be false. This also covers the thorny issue of _type substitution_, which we’ll discuss later.
* `IsUnknown` - determines whether this type is unknown, i.e. it cannot be properly presented.
* `IsValid()` - checks if the type refers to valid _declared elements_.
* `IsImplicitlyConvertibleTo()` and `IsExplicitlyConvertibleTo()` check whether this type is implicitly or explicitly convertible to some type (`IType`) via a _type conversion rule_. Type conversion rules are by themselves singleton classes that implement the `ITypeConversionRule` interface. For example, to use C# type conversion rule, one would use `CSharpTypeConversionRule.Instance`.

So where is `IExpressionType` used? Well, the most obvious example is if you’ve got an expression (i.e., an `IExpression`), you can use its `GetExpressionType()` method to get the expression’s `IExpressionType`. Of course, in most cases the expression type is expected to be an actual object type in which case it makes sense to call `IExpressionType.ToIType()` method. Of course, it returns `null` in case when the expression in question is not an `IType`.

Let us now take a look at the `IType` interface.

## IType

The `IType` interface is used to represent various type constructs - not just individual types, but constructs such as anonymous types, arrays, pointers, and others.

`IType` is also an interface, which has some members that we need to discuss:

* `GetPresentableName` - this returns the presentable name of the type for a particular language, such as C# or VB.NET. The string returned is ’presentable’ in the sense that the shortest and most concise string is used. For example, a 32-bit integer will be presented in C# as `int` and not `System.Int32`.
* `GetScalarType()` returns a _declared type_ that corresponds to the ’scalar’ portion of the type. So, if your `IType` happens to be `int *` (a pointer to an `int`), the returned `IDeclaredType` will correspond to `int`.
* `Classify` is a property that helps you determine if the type is a value or reference type. `null` is returned in case this cannot be determined.

Of all the interfaces that implement `IType`, the one we’re most interested in is `IDeclaredType`, so let’s discuss what it’s all about.

## IDeclaredType

The `IDeclaredType` is the interface type you’ll be seeing most when working with types. Essentially, an `IDeclaredType` is an interface for a type that might have a declaration (i.e., it might correspond to some `ITypeElement`). Examples of declared types might be `int` or `Foo`, provided you have a declaration of `Foo`, of course.

The interface’s most important member is `GetTypeElement()`, which may return a corresponding `ITypeElement` or `null` if one isn’t available. Other members include:

* `GetClrName()` - return a CLR name (`IClrTypeName`) of the type. This interface can then be used to get the short and full names of the type, to acquire lists of namespace names for this type, and so on.
* `GetSubstitution()` - returns the substition that is made for this type. Substitutions are mechanisms for replacing type parameters (i.e., generic paramers) with concrete types.
* `Resolve()` attempts to resolve this type. The resulting `IResolveResult`’s `IsValid()` method can be called to check if resolution succeeded.
* `IsSubtypeOf()` - determines whether this type is a subtype of some other `IDeclaredType`.

But that’s not all! In addition to the above members, there is also a `DeclaredTypeExtensions` class, which provides additional utility methods. For example, the `GetSuperTypes()` method returns an enumeration of `IDeclaredType`s corresponding to this type’s parents.

We’ve looked at a hierarchy of types, but how does this help us? Well, now that we know a little about types, we can start looking at yet another hierarchy -\- a hierarchy of _declared elements_.

## IDeclaredElement

![Type hierarchy for IDeclaredElement](ideclaredelement_hierarchy.png)

The `IDeclaredElement` is ReSharper’s über-interface for defining various _physical_ language constructs in ReSharper. By _physical_ we assume that a PSI tree has been built for them, in other words, we have the source code that defines this element. Declared elements are used to represent not just different constructs (such as a CLR class or a CSS function) but also different facets of structures.

Let us attempt a top-down overview of the `IDeclaredElement` interface and its implementers by looking first at its members.

* `GetDeclarations()` returns the declarations (i.e., `IDeclaration`s) that this declarated element contains. We’ll get to declarations in a while.
* `ShortName` returns a ‘friendly name’ of a declared element.
* `GetElementType` returns the type of the declared element. This value is typically language-related, i.e., for a C# element we’d get, for example, a `CSharpDeclaredElementType`.
* `PresentationLanguage` affects the language that is used to create this type.
* `GetSourceFiles()` returns a collection of `IPsiSourceFile`\` s that contain this type’s definition. (Remember that thanks to `partial` types, a type definition may span multiple files.)
* `HasDeclarationsIn()` lets you check whether or not any of the declared element’s declarations appear in a particular file.

Let’s drill down one level in the inheritance hierarchy and discuss the various declared element types.

## Declared Element Types

Most of the types under `IDeclaredElement` are interfaces which reflect platform-specific or language-specific declared elements. Currently, there are declared element types for CSS, HTML, JavaScript, Razor, CLR languages, and so on. There are also a few very specific declared elements such as `ILabel` that appear at this stage simply because there is no other suitable location for them.

In the context of this guide, we’re going to take a look at the `IClrDeclaredElement` interface which, predictably, reflects the declared elements for CLR (C#, VB.NET) languages.

## IClrDeclaredElement

This interface doesn’t contain much, its main responsibility being to delineate the fact that its implementers are CLR declared elements. Apart from yielding an `IPsiModule` handle, it can also return its containing type (if any) via `GetContainingType()` or containing type member via `GetContainingTypeMember()`.

This interface is interesting to us because of its immediate descendants. The ones worth mentioning are:

* `IAttributesOwner` - this interface is implemented by any declared element that can be decorated with attributes. You would typically use this interface to manipulate the attributes that some element has attached to it.
* `IParametersOwner` - if the declaring element takes parameters (note, we’re talking about _ordinary_ parameters, not type parameters), this interface helps us manipulate them.
* `ITypeOwner` - means the declared element has a type of its own.
* `ITypeParametersOwner` - means this declared element has type parameters. Examples would be classes and methods.

 > : Concrete declared elements can implement multiple interfaces. As a result, a single declared element can be both an `ITypeOwner` and an `ITypeParametersOwner`. The majority of declared elements implement multiple interfaces.
 >
 {type="note"}

## ITypeElement

While on the subject of _types_ as such, let’s discuss the CLR types and how they are exposed. Walking down the inheritance hierarchy, underneath `IClrDeclaredElement` we first encounter the `ITypeParametersOwner` interface which, as mentioned previously, is implemented by any construct that can have type parameters. This interface exposes just one property, a list of `ITypeParameter` objects that correspond to type parameters.

What’s more interesting, however, is the interface inheriting from it. This interface is called `ITypeElement`. What’s important about this interface is that it represents a CLR type, such as a class, structure, enumeration or delegate. This interface is inherited by interfaces such as `IClass`, `IStruct`, and so on. Let’s discuss some of its members:

* `NestedTypes` - this property is itself a list of `ITypeElement` entities representing the nested types.
* `Constructors`, `Operators`, `Methods`, and so on --\- all these properties yield enumerations of appropriate type members.
* `GetMembers()` - helps you get all of the above in a single list of `ITypeMember` entities.
* `GetContainingNamespace()` - does exactly what it says, and may return `null` if the type isn’t within a namespace (i.e., it is at global scope).

There is also a `TypeElementExtensions` class which provides additional functionality. For example, to check if a type is a descendant of another type (i.e., has another type above it somewhere in the inheritance chain), you can use the `IsDescendantOf()` method.
