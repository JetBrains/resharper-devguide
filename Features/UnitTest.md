# Unit Test Framework Support

Most of ReSharper plugins involve support for different unit test frameworks. This page describes ways in which you can add unit test support to your own framework.

## Overview

The first thing you need to understand about ReSharper testing is the way in which ReSharper actually _finds_ tests. One would expect that the way of finding tests in ReSharper is by simply traversing the PSI (i.e., the syntax tree from parsed source code) looking for the appropriate attributes. However, on large solutions, this is too slow, which is why another approach exists: traversing the compiled assemblies and acquiring test information from there.

As a result, plugin writers need to support two separate test analysers - the metadata explorer and the file explorer.

### Metadata Explorer

The metadata explorer is used to explore the metadata of compiled CLR assemblies to look for test elements. It is not applicable to non-CLR unit test frameworks such as QUnit.

In order to support the metadata explorer, plugin writers need to create a class decorated by the `\[ReSharper:MetadataUnitTestExplorer\]` interface and implementing the `IUnitTestMetadataExplorer` interface. This interface has two members:

* The `Provider` property, which returns the unit test provider.
* The `ExploreAssembly()` method, which is used to tell an explorer to explore the metadata for a particular assembly.

h3. File Explorer

The file explorer acts in a way similar to the metadata explorer, the only difference being that instead of an `ExploreAssembly()` method it has an `ExploreFile()` method which, predictably, takes an `IFile` to explore.

A typical implementation of the `ExploreFile()` method is to simply take the file and call `ProcessDescendants()` on it, passing your own file explorer class which implements the `IRecursiveElementProcessor` interface. From then on, the idea is to traverse the PSI tree of the file (just like you would if you were writing an ordinary analyzer), identify the elements which constitute test fixtures, tests, and so on, and then construct appropriate `IUnitTestElement` entities.

h3. IUnitTestElement

The `IUnitTestElement` interface is the core interface that is involved in unit testing. Essentially, this interface needs to be implemented by any class in your code that _represents_ a test.

Currently, ReSharper currently has several base classes for test elements, such as `MsTestElementBase` or `NUnitElementBase`. These elements are further subclassed by the _form_ in which a test element appears. Some examples are:

* A fixture
* A method
* A row or test case


Other elements can be created to suit your needs. For example, if your tests are defined in fields, you could create a `SomeFrameworkElementBase` that implements `IUnitTestElement` and subsequently derive from it your `SomeFrameworkFieldElement` class.

If you look at the existing implementations of `XxxElementBase` classes, you will notice that each one of them takes a `XxxTestProvider` as a parameter to be injected in the constructor. (This provider is also returned in the `Provider` property.) This provider class is the class which actually explores the assembly and creates the tie-in between ReSharper and the unit testing framework of choice.

h3. IUnitTestProvider

The unit test provider is a class which implements the `IUnitTestProvider` interface and is decorated with the `[ReSharper:UnitTestProvider]` attribute. The actual binding of the provider to a particular element happens in the `IsElementOfKind()` method. This method is used to determine whether the particular element is, in fact, an element of a particular kind. The possible kinds, which are part of the `UnitTestElementKind` enumeration, are as follows:

* Unknown -- we don’t know what this is
* Test -- the actual test which does does something. Here we mean something like a `[ReSharper:Test]` or a `[ReSharper:TestCase]` -- both count as tests.
* TestContainer -- this represents a container of a set of tests. Typically, this matches a `[ReSharper:TestFixture]`, but if we have a test with several rows/test cases, then this would also cover a `[ReSharper:Test]`.
* TestStuff -- this is any element which relates to tests, such as all of the aforementioned elements plus elements such as `[ReSharper:SetUpFixture]`


There are, in fact, two overloads of `IsElementKindOf()`. One takes an `IDeclaredElement`, whereas another atakes a `IUnitTestElement`. The overload that takes an `IUnitTestElement` is simple -- it simply implements a set of rules similar to the list above, for example:
{code:none}
public bool IsElementOfKind(IUnitTestElement element, UnitTestElementKind elementKind)
{
  switch (elementKind)
  {
    case UnitTestElementKind.Unknown:
      return !(element is NUnitElementBase);
    case UnitTestElementKind.Test:
      return element is NUnitTestElement || element is NUnitRowTestElement;
    case UnitTestElementKind.TestContainer:
      return element is NUnitTestFixtureElement || element is NUnitSetUpFixtureElement || (element is NUnitElementBase && !element.Children.IsEmpty()) || element is NUnitTestFromAbstractFixtureFakeElement;
    case UnitTestElementKind.TestStuff:
      return element is NUnitElementBase;
    default:
      throw new ArgumentOutOfRangeException("elementKind");
  }
}
{code}
The overload taking an IDeclaredElement is a bit more complicated. Essentially, this overload checks for the `UnitTestElementKind` on the actual code element. As a result, it is the responsibility of the developer to check that this is indeed the case.

Here’s an example. Let’s suppose that we want to determine whether something is a unit test. This would imply that our `IDeclaredElement` is:

* An `ITypeMember`
* An `IMethod`, since NUnit tests are kept in methods
* Is public and not abstract
* Is not generic
* Has any attribute (direct or derived) from a set containing `Test`, `TestCase`, etc.


In order to determine whether a particular attribute has been applied to a method, we create declarations similar to the following
{code:none}
IClrTypeName TestAttribute = new ClrTypeName("NUnit.Framework.TestAttribute");
{code}
and we subsequently check that the attribute owner (i.e. our method) has this attribute using the `HasInstanceAttribute()` method. Determining _derived_ types is a bit more complicated -- take a look at the `UnitTestAttributeCache` class in ReSharper for an illustration of how this is handled.

h3. RemoteRecursiveTaskRunner

We now come to what is arguably the most complicated part of all: the task runner which actually runs the unit tests. This class typically inherits from `RecursiveRemoteTaskRunner` and is expected to implement several methods from its parent types. Let’s take a look at some of them.

The `ConfigureAppDomain()` method is used to configure the `AppDomain` for running the tests. This method takes a `TaskAppDomainConfiguration` object and is expected to modify it, setting things like the priority or the apartment state. Some of these settings can be read from the configuration file (see `NUnitTaskRunner` for an example).

The `ExecuteRecursive()` method is the method that actually executes the tasks.

If you peek into the `NUnitTaskRunner`, you may see something strange: infrastructure code being acquired from resources and compiled before test execution. Please note that this approach is only necessary if you want one version of your plugin to support many versions of your unit test framework. (And, even if you do that, you will have problems because of changing APIs.) Thus, if you are making a plugin for your own unit test framework, your best bet is to simply keep the test framework and the plugin in sync, i.e. release plugins that correspond to your unit test framework versions

### Row Test Support

Most test frameworks provide support for parameterised tests, usually in the form of a test method that takes parameters, and is called multiple times by the test framework's runner. NUnit's parameterised tests look like this:

```cs
[TestCase(12, 3, 4)]
[TestCase(12, 2, 6)]
[TestCase(12, 4, 3)]
public void DivideTest(int n, int d, int q)
{
  Assert.AreEqual( q, n / d );
}
```

or like this:

```cs
[TestFixture]
public class MyTests
{
  [Test, TestCaseSource(typeof(MyFactoryClass),"TestCases")]
  public int DivideTest(int n, int d)
  {
    return n/d;
  }

  // ...
}

public class MyFactoryClass
{
  public static IEnumerable TestCases
  {
    get
    {
      yield return new TestCaseData(12, 3).Returns(4);
      yield return new TestCaseData(12, 2).Returns(6);
      yield return new TestCaseData(12, 4).Returns(3);
      yield return new TestCaseData(0, 0)
        .Throws(typeof(DivideByZeroException))
        .SetName("DivideByZero")
        .SetDescription("An exception is expected");
    }
  }
}
```

The main difference here is that in the first example, all rows are known at edit/compile time, while in the second example, the rows are essentially dynamic - the implementation of `TestCases` can return any number of rows, potentially different each time the test is run.

Row tests are handled by providing another implementation of `IUnitTestElement`. Conceptually, you would have a test fixture element that represented test classes. It's `Children` property contains test method elements, and each test method can optionally have any number of row test elements in its `Children` property. (Strictly speaking, ReSharper can support arbitrary levels of test elements, so you can represent assembly fixtures or other tasks that need to run before test classes, test methods and row tests).

The implementation of the row test element is very similar to the test method and test class elements:

* If you can identify the row test from the source, then you should add row tests when parsing the file looking for elements, just as you would for test classes and test methods. You should try and reuse elements so that ReSharper can maintain state in the test runner window (and especially important for multiple runs with dotCover). So, use a predictable id for the element, and check to see if it already exists by calling `IUnitTestElementManager.GetElementById`. If it does exist, update anything that needs updating (e.g. set the state to `UnitTestElementState.Valid`). If it doesn't exist, create it and give it to the element consumer while parsing the file.
* Make sure your `UnitTestElementComparer` knows how to compare row test elements.
* Make sure `IUnitTestProvider.IsElementOfKind` knows how to handle row test elements.
* Make sure `IUnitTestSerializer` knows how to serialise and deserialise row test elements.
* Make sure your row test element implements `IUnitTestElement.GetTaskSequence` to return a list of tasks representing how to run that particular row test. This essentially means calling the row test's `Parent.GetTaskSequence` and appending a new instance of a class that derives from `RemoteTask` and represents this row test. Again, this is important for dotCover code coverage.

In your test runner, when your test framework reports that a row test is executing, try to find the task instance, and report progress to ReSharper in the normal manner.

If you don't know the row tests in advance, but they are instead discovered at run time, you can still work with ReSharper. In your test runner, when you encounter a test that you don't have a task for, you create your row test `RemoteTask` instance and call `IRemoteTaskServer.CreateDynamicElement(task)`. Then use the task as normal to report that the test is starting, skipped, passed or failed. If you do have tasks available, make sure they are used, rather than always creating a new task. This is especially important when working with dotCover.

On the server side, ReSharper will try to create a new element for that task. It does this by checking to see if your `IUnitTestProvider` implements `IDynamicUnitTestProvider`. If so, it will call `IDynamicUnitTestProvider.GetDynamicElement`, passing in the task your runner created. It also passes in a dictionary of `RemoteTask` to `IUnitTestElement`, representing all of the tasks and elements that are present in the run. You should now:

* Using information in the `RemoteTask` your runner created, look for the parent task that represents the row test's test method in the dictionary of tasks. You will need to check the type of the `RemoteTask`, downcast and verify that the task represents the parent test method (e.g. you might need to check method name, type name and even assembly location). This will give you the task to lookup the `IUnitTestElement` of the row test's parent in the passed in dictionary.
* Get a read lock so that it is safe to read the PSI and project model ( `using(ReadLockCookie.Create())...` )
* Create the same predictable id for the row test element, and try to get it from the `IUnitTestElementManager.GetElementById`. If you get one, make sure to set the state to `UnitTestElement.Dynamic` and reset the `Parent` property to be the parent test method element. This is necessary for random row tests - it might not have been part of the last test run, so wouldn't have been included in the task sequence. However, it might have run previously, and the `UnitTestElementManager` might still have a handle to it, but might have marked it invalid, ready for clean up.
* If you don't get an existing element, create a new one, and return it. It is important that if you are creating a row test element due to a dynamic request, that the `IUnitTestElement.State` property is set to `UnitTestElementState.Dynamic`, so ReSharper knows how to clean the element up on subsequent runs.

