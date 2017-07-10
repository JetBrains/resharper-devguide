---
title: Generating Comments
---

Comments occupy a curious place within the ReSharper ecosystem. On the one hand, a comment may be a simple string encapsulated in a node of a physical tree. On the other hand, a comment preceding a method consist of valid XML that is subsequently represented as a proper data structure.

In this section, we’re going to talk about code comments, the ways in which they are represented and manipulated.

## Basic Comments

Before we begin, the most important thing to realize is that *basic comments (as opposed to XML comments) are only part of the physical tree*, i.e., comments are available in the tree structure that is the result of lexing of the source code, but they are _not available_ when dealing with the definitions that are the results of the parsing stage.

Consider the following declaration:

```csharp
class C
{
  // this is a comment
}
```

From the lexical perspective, the class body can be treated essentially as a sequence of tokens such as a left curly brace, newline, and so on. The comment above is represented by an `ICommentNode` structure with a `type` of `END_OF_LINE_COMMENT`. Were you to change the comment to `/* this is a comment */`, its type would be `C_STYLE_COMMENT`.

But how do you actually _get_ an `ICommentNode`? This depends on the type of the feature you’re workin on. For example, let’s say that you’re writing a context action which replaces any comment it finds with `/* hello, world */`. This context action would then have its `IsAvailable()` method defined as follows:

```csharp
public bool IsAvailable(IUserDataHolder cache)
{
  return provider.TokenAfterCaret is ICommentNode;
}
```

The provider’s `TokenBeforeCaret` and `TokenAfterCaret` properties yield the tokens that appear before and after the caret in the file being edited. Detecting them is easy, but what about actually editing them?

The bad news is that you cannot just call some `SetText()` method on a comment node and be done with it - instead, you need to create a brand new `ICommentNode` and replace the old one. Thus, the implementation of `ExecutePsiTransaction()` for our context action would look something like the following:

```csharp
protected override Action<ITextControl> ExecutePsiTransaction(ISolution solution, IProgressIndicator progress)
{
  var factory = CSharpElementFactory.GetInstance(provider.PsiModule);
  var oldComment = provider.TokenAfterCaret as ICommentNode;
  var newComment = factory.CreateComment("/* hello, world */");
  ModificationUtil.ReplaceChild(oldComment, newComment);
  return null;
}
```

In the above example, we use a `CSharpElementFactory` to manufacture a brand new comment, then use `ModificationUtil` to replace the old comment with the new one. If you wanted to simply get rid of the comment, you could use `ModificationUtil.DeleteChild()` instead.

## XML Doc Comments

Now, what would happen if you declared the comment as `/// <z>this is a comment</z>`? The end result in this case is that the comment has a type of `DocComment`. A ‘doc comment’ is different though - unlike an ordinary comment, a doc comment is typically attached to a particular element of the ‘chemical’ tree, such as a class or class member (field, property, etc.). It’s also possible in certain cases for a doc comment to span _several_ elements at once. Here is an example.

```csharp
class C
{
  /// <summary>Co-ordinates</summary>
  private int a, b;
}
```

While the above may not be a best example in terms of programming, it does illustrate the fact that even a doc comment can affect more than one class member though, of course, in terms of the chemical tree, the two declared fields are collected under the umbrella of an `IMultipleFieldDeclaration`.

So now let’s get back to the usual questions. First, how do you determine that the caret is on a doc comment? The same way as before, except that the type of the node is different:

```csharp
bool isOnDocComment = provider.TokenAfterCaret is IDocCommentNode;
```

Now, you could always try and replace the doc comment wholesale (just like we did with simple comments), but instead of calling `CreateComment()` on a `CSharpElementFactory`, you have two separate methods that you can call for creating doc comments:

* `CreateDocComment()` expects no line breaks in the specified parameter and simply prefixes the text with a `///`.
* `CreateDocCommentBlock()` actually splits the parameters you provide into separate lines and then adds each one of them as part of a large block. Each line is, of course, prefixed with `///`.

The above methods may appear useful, but they have nothing to do with XML whatsoever. If you’re after manipulating XML, there’s an entirely different set of structures that you have to use.

### XML Doc Comment Editing

Replacing the doc comment block completely is a pretty bad idea, because in most cases it’s up to you to keep the XML structure consistent. As a result, there’s a better way.

First of all, it’s worth explaining what the _block_ part in `XmlCommentBlock` means. Essentially, a block is simply a collection of XML comment nodes that sit next to one another. For example, you might have a `<summary>` on one line and a `<param>` on another: these lines together form part of a block.

A single doc comment is then part of that block. For example, a `<summary>` would be a separate `IDocCommentNode` that is part of a larger `IDocCommentBlockNode`. Please note that a `IDocCommentBlockNode` is created even if there is only one `DocComment` in it.

Unlike basic comments, doc comment blocks are language-specific. This means that, if you’re working with C#, you’ll most likely be working with an `ICSharpDocCommentBlockNode`. This type of node is a lot more interesting because, unlike the typical XML doc-related API which would yield you a simple `XmlNode` (see e.g., the `IDeclaration.GetXmlDoc()` method), the `ICSharpDocCommentBlockNode` can yield you a fully fledged XML PSI interface:

```csharp
var node = provider.TokenAfterCaret.Parent as ICSharpDocCommentBlockNode;
IDocCommentXmlPsi xmlPsi = node.GetXmlPsi();
```

Note the use of `Parent` property in the above call: your caret is, essentially, on a doc comment node, whose _parent_ is the block.

The great thing about `IDocCommentXmlPsi` is that it contains a myriad of utility methods for adding or modifying particular XML doc declarations such as summary, parameters, exception information, and so on. For example, the following piece of code lets you add a `<summary>` to the block comment:

```csharp
var factory = XmlElementFactory.GetInstance<XmlDocLanguage>();
var summaryTag = factory.CreateTag("<summary>this method rocks</summary>");
xmlPsi.AddSummaryNode(summaryTag);
```

Please note that the XML PSI interface is only available on _existing_ XML doc comment blocks - you cannot get this interface if you have an ordinary comment `/// like this one` in your code. Thus, if you need to create an XML doc comment block from scratch, you should first use a `CSharpElementFactory` to create the initial content for the block, and then get the `IDocCommentXmlPsi` interface from it.

