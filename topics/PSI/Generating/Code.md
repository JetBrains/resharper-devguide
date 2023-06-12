[//]: # (title: Generating Code)

If you plan to write plug-ins that generate code, it's important to understand how to use ReSharper to create new code elements and insert them into existing types.

Depending on the situation, you will most likely encounter two usage patterns:

* One element needs to be replaced by another. This is typically achieved by either calling the `ReplaceBy()` method on the element to be replaced or, if this method is not available, by physically removing one element and adding another.
* An element needs to be inserted as a child of another element. In this case, there is typically a method in the API for doing exactly that.

## Element factory

The Element Factory is a class which is used to manufacture all types of entities, be it statements or whole declarations. The element factory is created via a `GetInstance()` method, which is supplied the corresponding `PsiModule`:

```csharp
var factory = CSharpElementFactory.GetInstance(provider.PsiModule);
```

Having acquired the element factory, you can now use it to manufacture various objects by invoking one of its methods. For example, to create a simple statement, you would write the following simple piece of code:

```csharp
ICSharpStatement st = factory.CreateStatement("int x = 0;", EmptyArray<object>.Instance);
```

You're probably curious to know what the second parameter is. Essentially, the second parameter is an array of `object` which act in a similar way to the way the `args` parameters act in a `string.Format()` call.

Why are these parameters needed? Well, interestingly enough, the parameters allow you to specify additional checks that need to be made during the generation of code. For example, let us suppose that we need to create a statement similar to `int x = 0;`, but the name of the variable is provided elsewhere. What would happen if that variable's name was `int` or `event`? Creating the statement would result in erroneous code.

But consider the following adjustment:

```csharp
ICSharpStatement st = factory.CreateStatement("int $0 = 0;", new object[]{varEvent});
```

Now we're passing the variable as a parameter, with the dollar `$n` notation used to specify the `n`th argument to be inserted. If `varEvent` denotes an identifier of `event`, the generated C# statement would now be:

```csharp
int @event = 0;
```

## Type factory

Creating types is an important task in the ReSharper ecosystem. For example, say you wanted to write a Context Action to implement change notification in a class. This would require you to create, say, an `INotifyPropertyChanged` interface type.

Types are created using the `TypeFactory`. There are many overloads for creating types. The simplest simply creates a type when provided the type's FQN (fully qualified name):

```csharp
IDeclaredType propertyChangedInterface =
  TypeFactory.CreateTypeByCLRName("System.ComponentModel.INotifyPropertyChanged", provider.PsiModule);
```

Creating a generic type is a little bit more complicated. You first use the `TypeElementUtil` helper class to get the type element for the generic type. The 'backquote notation' is used for this, so that `Lazy<T>` effectively becomes ```Lazy`1```. After you have received the type element, you call the `TypeFactory` method to create the type, providing first the type element you created, and then an array of types that specify the generic arguments:

```csharp
var lazyTypeElem = TypeElementUtil.GetTypeElementByClrName(new ClrTypeName("System.Lazy`1"), psiModule);
var lazyType = TypeFactory.CreateType(lazyTypeElem, thisType);
```

Please note that if you want to use the type to create, e.g., statements with the `$` notation, you cannot simply use the parameter of `IDeclaredType` - you need to call `GetTypeElement()` on it.
