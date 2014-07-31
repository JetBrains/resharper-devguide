# Daemons

As you may have noticed, ReSharper features background analysis of your code. In fact, at any one time there are several different analyzers running at different _stages_, all combining to provide the most complete assessment of your code.

## Daemon Stage

When writing a plugin, you will most likely be implementing your own daemon stage - a class decorated with the `DaemonStage` attribute that implements the `IDaemonStage` interface.

The interface is fairly simple, and has two members:

* The `CreateProcess()` method is where one or more daemon stage processes are created. A daemon stage process is created on a per-document basis to analyze files and provide information about them. Note that `IDaemonStageProcess` can yield several different processes that can correspond to different injected PSI documents. 
* The `NeedsErrorStripe()` method is used to determine whether or not analysese from this daemon stage contribute to the markers on the right-hand stripe, as well as to the list of document errors. There are only three possible values from the `ErrorStripeRequest` enumeration that can be returned here.

ReSharper also comes with concrete base classes that implement `IDaemonStage` (e.g., `CSharpDaemonStageBase`) - these reduce the `CreateProcess()` method to creating a single `IDaemonStageProcess` corresponding to a single PSI file.

## Daemon Stage Process

The daemon stage process also happens to be a class that implements the `IDaemonStageProcess` interface. Typically, this class is passed the daemon stage so that it can get, e.g., the solution or current file from that variable.

The `IDaemonStageProcess` interface has two members:

* A property called `DaemonProcess` which returns the corresponding daemon process. Typically you would return the value that you passed into the constructor.
* A method called `Execute()` where the actual analysis happens.

Let's talk about the Execute method in more detail. If you've inherited from a class such as `CSharpDaemonStageBase`, your `CreateProcess()` override already has a parameter of type `ICSharpFile` that you can use. If not, you can use an extension method in `PsiManagerExtensions` to acquire the file:

```cs
var file = process.SourceFile.GetPsiFiles<CSharpLanguage>();
```

Now that you have the code structure you want to analyse, you spin up one of the element processors on it. An element processor is a class implementing an interface such as `IRecursiveElementProcessor`, that lets you walk around all the tree nodes in a particular block of code. In the case of a recursive element processor, it has the following methods:

* `InteriorShouldBeProcessed()` determines whether the interior of the provided ITreeNode should be processed. If it returns true, the processor will drill down into the inner structures. If it returns false, the processor will stop at this level.
* `ProcessBeforeInterior()` lets you process the tree node before you've gone into its descendants.
* `ProcessAfterInterior()` lets you process the tree node after you've gone through its descendants.

## Highlighting

Now, the end result of all the work done by the recursive element processor is a set of _highlighting_. A highlighting is essentially information about a piece of code that has to be highlighted (underlined with a wavy line), as well as shown in the marker bar and possibly in the document errors. Highlightings also happen to be the points on which Quick-Fixes work.

Highlightings all implement the `IHighlighting` interface and are decorated with one of the highlighting attributes related to the type of highlighting it is. There are two types of highlightings available:

* The `StaticSeverityHighlighting` attribute is used to indicate a highlighting that never goes away. This means that when you set it, it _will_ be shown.
* The `ConfigurableSeverityHighlighting` lets the user configure if and when this highlighting is shown. As a consequence, it has context actions to let you disable it temporarily or edit its settings.

There are several pieces of information that your own highlighting class can provide. At a minimum, you can specify:

* The severity of the highlighting. This is defined in the severity attribute.
* The tooltip for the highlighting when you move the caret over it in the editor.
* The tooltip for the marker bar.

So - coming back to the daemon stage process - you have invoked the element processor (or several) and got a set of highlightings. The last thing to do is to commit the results using the commiter that is provided as a parameter to the `Execute()` method:

```cs
commiter(new DaemonStageResult(elementProcessor.Highlightings));
```

And that's it - your daemon is ready to go!

## ElementProblemAnalyzer

If you are looking for a simpler way to perform an analysis and add highlightings to code, the `ElementProblemAnalyzer<T>` class presents an alternative. If you use this class, you only have to have one single type instead of defining both a daemon and a daemon stage.

Put briefly, the element problem analyzer is a class that:

* Inherits from `ElementProblemAnalyzer<T>` where `T` is a type inheriting from `ITreeNode` (e.g., an `IExpression`) that will subsequently be subject to analysis throughout the file.
* Is decorated by the `[ElementProblemAnalyzer]` attribute. This attribute takes parameters which indicate the elements the analyzer highlights as well as the highlighting types it uses.
* Implements a single method called `Run()` where analysis and highlighting happens.

The `Run()` method is invoked recursively for every element of type `T` found in the file. For example, if `T == IExpression`, the `Run()` method will be called for every expression that is found. The method signature is as follows:

```cs
void Run(T element, ElementProblemAnalyzerData data, IHighlightingConsumer consumer)
```

The parameters are:

* `T element` - the element currently being analyzed. Its type is defined when inheriting the class.
* `ElementProblemAnalyzerData data` - contains references to certain useful parts of the ecosystem, such as the controlling `IDaemonProcess` and the settings store.
* `IHighlightingConsumer` - this interface is used to add the highlightings to various elements.

The following is an example implementation of an element problem analyzer:

```cs
[ElementProblemAnalyzer(new[] { typeof(IExpression) }, 
  HighlightingTypes = new[] { typeof(UseOfInt64MaxValueLiteralHighlighting) })]
public class UseOfInt64MaxValueLiteralChecker : ElementProblemAnalyzer<IExpression>
{
  protected override void Run(IExpression element, ElementProblemAnalyzerData data, IHighlightingConsumer consumer)
  {
    var value = element.ConstantValue;
    if (!value.IsLong())
      return;

    long number = Convert.ToInt64(value.Value);
    if (number != Int64.MaxValue)
      return;

    consumer.AddHighlighting(new UseOfInt64MaxValueLiteralHighlighting(element), 
      element.GetDocumentRange(), element.GetContainingFile());
  }
}
```

