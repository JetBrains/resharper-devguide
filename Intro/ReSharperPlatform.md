# What is the ReSharper Platform?

The ReSharper Platform is an Open API for building smart IDE features and developer tools. The platform has a deep understanding of your solution. By parsing the files in your projects, ReSharper can build and cache a syntactic and semantic model of your code, which can be used to perform analysis, showing highlights inline in the IDE as you type. The syntax trees can be manipulated in memory to provide safe refactoring, and a powerful cross-language reference system allows for very rich navigation, search and usage analysis.

The ReSharper product is the main client and extension to the ReSharper Platform. The base platform provides services such as Visual Studio integration, project model abstractions and the infrastructure to work with abstract syntax trees and semantic models. The ReSharper product adds many user facing features on top of this, including different language support, from C# to JavaScript to HTML and CSS, to rich refactoring and unit testing support.

The ReSharper Platform is also very extensible. The whole API is public, and is written with the Open/Closed Principle very much in mind. Both the Platform and the ReSharper product can be extended, to provide new analyses, warnings and fixes, support for new unit test frameworks, and new end-user features, or by supporting new languages, and creating new products and feature sets.

The Platform integrates with Visual Studio, hosting products such as ReSharper, dotTrace and dotCover. But it can also be run standalone, hosting products with their own UI, such as dotPeek, dotMemory and the ReSharper command line tools.

