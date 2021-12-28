[//]: # (title: Custom Languages)

In this part of the guide we'll look at developing ReSharper support for a new language. We shall take a look at the following:



## Introduction

ReSharper supports a wide variety of language and also supports a mixture of languages (e.g., `.cshtml` files are a mixture of C# and HTML). Plugin writers can use existing infrastructure in order to support new languages within ReSharper. A language implementation can be a plugin or part of a plugin, and in most cases no special action is required for it to be picked up and recognized by ReSharper.

 >  While new language support may be provided in any .NET-compatible programming language, the provided parsing/lexing tools support only the C# programming language.
 >
 {type="note"}

## Tools

The parsing and lexing of languages requires specialized tools. This is why, as of the 7.0 release, the ReSharper SDK comes with a set of tools and target files as well as a fully worked-out example of supporting a new language. The following items are included:

* **CsLex** - a tool for creating lexical analysers for different languages.
* **Java** - contains IKVM infrastructure which permits the running of the Java-based parser generator.
* **MSBuild** - contains build tasks that can be used to automate lexer and parser construction.
* **parserGen** - a tool for creating ReSharper-compatible parsers.

In addition to the tools, the SDK also comes with a full language plugin example located in the `Samples/PsiPlugin` folder. This sample is a full implementation of a language plugin for supporting `.psi` files, which are parser definition files used by `parserGen`. As a result, plugin developers interested in using `parserGen` for new language development are advised to compile and install `PsiPlugin`, which can greatly simplify the process of working with parser definition files.

Let's go through the tools one by one, since all of them are critical in putting together a language plugin.

## CsLex

`CsLex` is a [freely available](http://www.cybercom.net/~zbrad/DotNet/Lex/) lexical analyzer generator that comes packaged in the SDK. CsLex can be found in the `Tools/CsLex` folder, which includes the following files:

* `CsLex.Targets` contains a set of targets that need to be included in the `.csproj` file using an `<Import>` directive:

```xml
<Import Project="$(ReSharperSdkTools)\CsLex\CsLex.Targets" />
```

* `CsLex.Tasks` contains the definition for a `CsLex` build task. It is automatically included by `CsLex.Targets`
* `lex.exe` is the executable that generates the lexical definitions. `lex.xml` is the XML documentation for the tool. You do not need to interact directly with these two files, as the build task takes care of that for you.

In order to generate lexical structures, your project needs to have a `.lex` file. You can learn more about the format of lexical definition files in the [CsLex documentation page](http://www.cybercom.net/~zbrad/DotNet/Lex/Lex.htm), but your definition also needs to have ReSharper-specific elements. To learn more about these elements, please take a look at the `psi.lex` file in the sample `PsiPlugin` that comes with the SDK.

The `.lex` file is compiled with a build action of `CsLex`. After compilation, CsLex generates from it a `_lex.cs` file that contains a generated lexer.

One of the building blocks of the lexer is the unicode definition file `Unicode.lex`. While this file currently lives in the `Tools/parserGen` folder, its purpose is to provide detailed information about various unicode rangers that the lexer can consume to correctly distinguish identifiers. In order to use this file, it needs to be copied into the `/obj` folder inside the project. The simplest way to do this is to add the following MSBuild directive to the project:

```xml
<Target Name="BeforeBuild">
  <MakeDir Directories="$(MSBuildProjectDirectory)\obj" />
  <Copy SourceFiles="$(ReSharperSdkTools)\parserGen\Unicode.lex" DestinationFolder="$(MSBuildProjectDirectory)\obj" />
</Target>
```

## ParserGen

`parserGen` is a parser generator that is used to define parsers for various languages. It lives in the `tools/parserGen` folder and happens to be written in Java, which is why there is also a `tools/Java` folder in the SDK that provides IKVM bindings. Let's go briefly through some of the *non-java* files that parserGen comes with:

* `ParserGen.Targets` and `ParserGen.Tasks` contain the MSBuild targets and build task respectively. Plugin writers need to add the following to the `.csproj` file:

```xml
<Import Project="$(ReSharperSdkTools)\parserGen\ParserGen.Targets" />
```

* `ParserGenTask.dll` provides a .NET shim for using a Java-based `ParserGen` build task.
* `Unicode.lex` is a file that actually relates to lexer construction. (See above.)

A parser is defined in a proprietary format in a file with a `.psi` extension and a specified build task of `ParserGen`. This is precisely the format for which `PsiPlugin` provides support, and thus you should compile and install `PsiPlugin` if you intend to work with `.psi` files. While no formal documentation exists, we recommend that you look at the `psi.psi` grammar file in `PsiPlugin` for a detailed example of a grammar definition.

## Language Definition

First of all, you need to create a class inheriting from `KnownLanguage`. The constraints on this class (let's call it `MyLanguage` here) are that it should:

* Be declared `public`
* Have one public static non-`readonly`, non-initialized field of a `MyLanguage` type
* Must have a parameterless constructor
* Should have a set of `protected` constructors to allow language inheritance.

The `MyLanguage` class must also be marked with the `[LanguageDefinition]` attribute. This attribute takes two parameters:

* The first and only required parameter is the name of the language.
* _(Optional)_ The `Edition` parameter specifies the ReSharper edition that this language supports. If you need to constrain the language to a particular R# edition, use an element of the `ReSharperEditions.Ids` enumeration here. Reminder: R# currently comes in three editions - C#, VB.NET and Full.

Here is an example of a language definition for the C# language:

```csharp
[LanguageDefinition(Name, Edition = ReSharperEditions.Ids.Csharp)]
public class CSharpLanguage : KnownLanguage
{
  public new const string Name = "CSHARP";

  [CanBeNull] public static readonly CSharpLanguage Instance;

  private CSharpLanguage() : base(Name, "C#") { }

  protected CSharpLanguage([NotNull] string name) : base(name) { }

  protected CSharpLanguage([NotNull] string name, [NotNull] string presentableName) : base(name, presentableName) { }
}
```

## Project File Type

Now, we need to tell ReSharper how to support a project of a particular type. This is another class that contains primarily metadata, and is used mainly to indicate the file extensions that correspond to a particular language. Just like the Language Definition class, this class (let's call it `MyLanguageProjectFileType`) requires that:

* It is declared `public`
* It has a `public`, `static`, non-`readonly` field of type `MyLanguageProjectFileType`
* It has a parameterless constructor. This constructor calls its base class' constructor, passing the array of file extensions that this project file type supports.
* It has a set of `protected` methods for inheritance

In addition, the project file type class has to be decorated with the `ProjectFileTypeDefinition` attribute. This attribute has the following parameters:
* The `Type` parameter requires name of the language (as per language definition)
* _(Optional)_ The `Edition` parameter determines the R# edition that supports this project file type.
* _(Optional)_ The `Internal` boolean parameter determines whether this project file type is only supported in internal mode. Plugin writers should not define this parameter.

Here is an example of a project file type class for HTML files:

```csharp
[ProjectFileTypeDefinition(Name)]
public class HtmlProjectFileType : KnownProjectFileType
{
  public new const string Name = "HTML";
  public new static readonly HtmlProjectFileType Instance;

  private HtmlProjectFileType() : base(Name, "Html", new[] {HTML_EXTENSION, HTM_EXTENSION})  {  }

  protected HtmlProjectFileType(string name) : base(name)  {  }

  protected HtmlProjectFileType(string name, string presentableName) : base(name, resentableName)  {  }

  protected HtmlProjectFileType(string name, string presentableName, IEnumerable<string> extensions) : base(name, presentableName, extensions)  {  }

  public const string HTML_EXTENSION = ".html";
  public const string HTM_EXTENSION = ".htm";
}
```

## Project File Language Service

Let's create a project file language service. This entity is used to tell us which language tree needs to be constructed for a particular file. For example, the `XamlProjectFileLanguageService` would create a XAML tree of the XAML file is part of the project, and only an XML tree if it is not.

The project file language service is a class which implements the `IProjectFileLanguageService` interface and is decorated with the `ProjectFileType` attribute relating to the project file type. The attribute takes a single parameter - the `typeof(MyLanguageProjectFileType)` that we created earler.

We are now getting in the thick of language development, this being the last stop before we go to define the overall language service (the one that exposes lexers, parsers and other supplementary information). Let us go through the members of our `MyProjectFileLanguageService` implementation.

* First of all, we need to have a public constructor that will take a parameter of the `MyProjectFileType` type. We'll need to store this parameter for returning it later.
* The `LanguageType` property is precisely the location where we would typically return the above stored value.
* The `Icon` property needs to return the icon for this type of file. If you are storing the icon internally as an embedded resource, use `ImageLoader.GetImage("icon.resource.name", null)` to return the icon.
* The `GetPsiLanguageType()` method takes an `IProjectFile` parameter. If the `LanguageType` property of this parameter matches our language, then we return the `Instance` field of our language. Otherwise, we return `UnknownLanguage.Instance`. Here is an example:

    ```csharp
    public PsiLanguageType GetPsiLanguageType(IProjectFile projectFile)
    {
      if (projectFile.LanguageType.Is<MyLanguageProjectFileType>())
        return MyLanguage.Instance;
      else
        return UnknownLanguage.Instance;
    }
    ```

* There is also an overload of the `GetPsiLanguageType()` method that takes a `ProjectFileType` as a parameter. The implementation of this method is similar to the one above:

    ```csharp
    public PsiLanguageType GetPsiLanguageType(ProjectFileType languageType)
    {
      if (languageType.Is<MyLanguageProjectFileType>())
        return MyLanguage.Instance;
      else
        return UnknownLanguage.Instance;
    }
    ```

* The `GetMixedLexerFactory()` method returns the mixed lexer factory. The mixed lexer (_mixed_ refers to the possibility of having mixed languages in a file) is returned by the language service, which we haven't defined. The typical implementation of this method is as follows:

    ```csharp
    public ILexerFactory GetMixedLexerFactory(IBuffer buffer, IPsiSourceFile sourceFile, PsiManager manager)
    {
      return MyLanguage.Instance.LanguageService().GetPrimaryLexerFactory();
    }
    ```

 >  In the above code, `LanguageService()` is an extension method that resides in the `PsiLanguageTypeExtensions` class.
 >
 {type="note"}

* The `GetPreprocessorDefines()` method is used to return a set of `PreProcessingDirective` definitions. If your language doesn’t have preprocessing directives, in which case you can simply return `EmptyArray<PreProcessingDirective>.Instance`.
* The `GetPsiProperties()` method is used to return a new instance of the PSI properties type for this file as follows:

    ```csharp
    public IPsiSourceFileProperties GetPsiProperties(IProjectFile projectFile, IPsiSourceFile sourceFile)
    {
      Assertion.Assert(projectFile.LanguageType.IsProjectFileType(LanguageType), "projectFile.LanguageType == LanguageType");
      return new MyLanguagePsiProperties(projectFile, sourceFile);
    }
    ```

We haven't seen the PSI Properties class, so that's coming up next.

## PSI Properties

Yet another service entity that is required for successful language support is the PSI Properties entity. In order to understand it why it is needed, it's important to understand that when working with files, we essentially operate on two different entities: `IProjectFile` and `IPsiSourceFile`:

* `IProjectFile` is part of the project model, i.e., it holds information about the file in the context of the project it's in. Among other things, it yields information regarding its `ProjectFileType` (we have defined this earlier), and also yields an `IProjectFileProperties` value that contains the types of file properties you see when you open the Properties window (or press F4) in Visual Studio.
* `IPsiSourceFile` is part of the PSI. It also yields a `ProjectFileType` as one of its properties, but it also yields a language (a `PsiLanguageType` inheritor such as `MyLanguage`) as well as a set of `IPsiSourceFileProperties`. These properties correspond to the code model, i.e., the AST that represents the file.

Thus, the PSI Properties entity is a kind of glue that manages takes as parameters both an `IProjectFile` and a `IPsiSourceFile` and manages to bind the two together.

The PSI Properties class is a class deriving from `DefaultPsiProjectFileProperties`, and is often located as an inner class of the above language service type. This class takes two parameters - the project file and the PSI source file. Typically, its constructor simply passes them up to the parent:

```csharp
private class MyLanguageFileProperties : DefaultPsiProjectFileProperties
{
  public MyLanguageFileProperties(IProjectFile projectFile, IPsiSourceFile sourceFile)
    : base(projectFile, sourceFile)
  {
  }
}
```

The above is a minimal implementation. Often, the PSI Properties type contains overrides for some of the properties defined by the base class. Here are a few such properties:

* `ShouldBuildPsi` determines whether the PSI tree needs to be built for this file. By default, the value is `true` for any type that isn't `null` or unknown. This property should be overridden in certain cases where it's not worth building a PSI - for example, in the case where you have a XAML file that's not part of a project.
* `ProvidesCodeModel` determines whether the file provides a code model. Not all files do - for example, an XML file does not. By default, this has the value of `ShouldBuildPsi`.
* `IsNonUserFile` determines whether this is a file is owned by the user or not. Typically this has the value of `\!IsCompile`, i.e. depends on the file's build action.
* `IsGeneratedFile` determines whether this file is generated or not. By default, R# tries to get this information from the provided `projectFile`.

## Language Service

We are finally ready to create is a language service -- an entity that finally lets us work with the language AST. This is a class inheriting from `LanguageService`. The requirements for this class are:

* It must be declared `public`
* It must have a public constructor taking _at least_ the language definition (i.e., the `MyLanguage` type) and an `IConstantValueService`.
* It must call the base constructor with the above parameters.

 >  After implementing the above, check that your breakpoints actually fire when opening the file with your chosen extension. An empty implementation of the above should be sufficient for ReSharper to identify the feature-related language service.
 >
 {type="note"}

The language service is where all the activity occurs. In particular, the language services exposes both the lexer and the parser, as well as a number of complementary services that may or may not be required for the particular language.

We begin our language implementation with the lexer. The language service type has two members relating to the creation of lexers.

The first is the `GetPrimaryLexerFactory()` method. This method returns a lexer factory, which is simply a class that implements the `ILexerFactory` interface and whose `CreateLexer()` method returns an instance of a lexer (which we'll define in a moment):

```csharp
private class MyLanguageLexerFactory : ILexerFactory
{
  public ILexer CreateLexer(IBuffer buffer)
  {
    return new MyLanguageLexer(buffer);
  }
}
```

Thus, the implementation of `MyLanguageService.GetPrimaryLexerFactory()` reduces to:

```csharp
public override ILexerFactory GetPrimaryLexerFactory()
{
  return new MyLanguageLexerFactory();
}
```

Of course, we have not yet mentioned the `ILexer`. Before we get on to actually making a lexer, it's also worth mentioning another `LanguageService` member - the `CreateFilteringLexer()` method. Now, just as the name suggests, a filtering lexer is a lexer that filters (i.e. ignores) certain token types. A filtering lexer inherits from the `FilteringLexer` class. Apart from having to have an `ILexer` as a constructor parameter passed up to its base class, it has a single method called `Skip()` that you need to override.

The `Skip()` method is simple: it takes a `TokenNodeType` and determines whether this is the kind of token that needs to be skipped. Typical tokens to be skipped often include whitespace, line breaks, comments or code within certain preprocessor directives. Now, the simplest way to provide support for this is to create a `NodeTypeSet` of all the tokens that you intend to skip as follows:

```csharp
internal static readonly NodeTypeSet TokensToSkip = new NodeTypeSet(
  new NodeType[]
    {
      MyLanguageTokenType.WHITE_SPACE,
      MyLanguageTokenType.NEW_LINE
    }
  );
```

With this definition in place, the implementation of the `Skip()` method can be as follows:

```csharp
protected override bool Skip(TokenNodeType tokenType)
{
  return MyLanguageService.TokensToSkip[tokenType];
}
```