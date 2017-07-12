---
title: Get a Tree Node by Full CLR Name
---

**What you should know beforehand:**
* [Project model](/HowTo/NavigateCode/NavigateCode.md#project-model-basics)
* [PSI](/HowTo/NavigateCode/NavigateCode.md#psi-basics)
* ShellLocks

**Examples ([?](HowTo.md#sample-solution)):**
* [PsiExtensionMethods.cs](https://github.com/JetBrains/sample-resharper-plugin/blob/master/SampleReSharperPlugin/src/PsiNavigation/PsiExtensionMethods.cs)

Typically, finding a node by a full CLR name is not a typical type of task for a plugin developer. This example is given mostly to help you better understand ReSharper's project model and PSI principles.

Let's imagine, you know a full CLR name of a class and want to navigate to it. See the example below.

1. First of all, we need to obtain an `IProject` instance for a particular project (by project's name). Here we work on the ReSharper's [project model](NavigateCode.md#project-model-basics) level. Let's make it an extension method of `ISolution`:

    ```csharp
    [CanBeNull]
    public static IProject GetProjectByName(this ISolution solution, string projectName)
    {
        var projects = solution.GetTopLevelProjects();
        return projects.FirstOrDefault(project => project.Name == projectName);
    }
    ```

    Here `GetTopLevelProjects()` returns a collection of projects in the solution.
1. Knowing the project, we can obtain a particular C# file from it (we're still on the project model level).

    ```csharp
    [CanBeNull]
    public static ICSharpFile GetCSharpFile(this IProject project, string filename)
    {
        var file = project.GetPsiSourceFileInProject(FileSystemPath.Parse(filename));
        return file?.GetPsiFiles<CSharpLanguage>().SafeOfType<ICSharpFile>().SingleOrDefault();
    }
    ```

    Here:
    * `GetPsiSourcFileInProject` returns the `IPsiSourceFile` by file name.
    * `GetPsiFiles` returns the collection of `IFile` (the main PSI element) that is then casted to `ICSharpFile` (`SafeOfType` is a safer sibling of `Enumerable.OfType<>()`). 
 1. On this step we work only with the PSI syntax tree (so, now, we're on the [PSI model](NavigateCode.md#psi-basics) level). Here we obtain an instance of `ICSharpFile` and find the required `ITreeNode` in it.

    ```csharp
    [CanBeNull]
    public static ITreeNode GetTypeTreeNodeByFqn(this ICSharpFile file, string typeName)
    {
        var namespaceName = GetLongNameFromFqn(typeName);
        var shortName = GetShortNameFromFqn(typeName);            
     
        var namespaceDecls = file.NamespaceDeclarationsEnumerable;
        var namespaceDecl = (from decl in namespaceDecls
            where decl.DeclaredName == namespaceName
            select decl).FirstOrDefault();
     
        if (namespaceDecl == null) return null;
        var typeDecls = namespaceDecl.TypeDeclarationsEnumerable;
     
        var resultList = (from node in typeDecls
            where node.DeclaredName == shortName
            select node).ToList();
     
        return resultList.FirstOrDefault();
    }
     
    private static string GetShortNameFromFqn(string fqn)
    {
        var pos = fqn.LastIndexOf(".", StringComparison.Ordinal) + 1;
        return pos > 0 ? fqn.Substring(pos) : fqn;
    }
     
    private static string GetLongNameFromFqn(string fqn)
    {
        var pos = fqn.LastIndexOf(".", StringComparison.Ordinal) + 1;
        return pos > 0 ? fqn.Substring(0, pos - 1) : fqn;
    }
    ```

    Here:
    * `GetShortNameFromFqn` and `GetLongNameFromFqn` are used to get namespace name and type's short name. Note that these functions are far from optimal (e.g., they do not take into account nested types).
    * `NamespaceDeclarationsEnumerable` returns a special enumerable type that allows to get all namespaces in a file.
    * `TypeDeclarationsEnumerable` returns all type declarations made within a namespace.
    * `ICSharpTypeDeclaration` provides the `DeclaredName` property that returns type's short name.
1. Finally, we can use our `GetTypeTreeNodeByFqn` function to navigate to the type declaration by knowing type's fully qualified name:

    ```csharp
    public static void NavigateToTypeNodeByFqn(this ISolution solution, string projectName, string fileName, string typeName)
    {
        solution.Locks.TryExecuteWithReadLock(() =>
        {
            var project = solution.GetProjectByName(projectName);
            var file = project.GetCSharpFile(fileName);
            var node = file.GetTypeTreeNodeByFqn(typeName);
            node.NavigateToTreeNode(true);
        });            
    }
    ```

    Here:
    * Note that to prevent conflicts with user input, we must obtain the read lock before calculating the node under the caret. Here this is done by means of the special method of the `IShellLocks` interface: `TryExecuteWithReadLock(...)`. The main difference comparing to the `QueueReadLock` method used in other examples is that is executed synchronously - as you can see, there's no `Lifetime` instance passed to the method.
