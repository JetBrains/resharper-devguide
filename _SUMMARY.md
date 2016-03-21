# Summary

* [Introduction](README.md)
* [The ReSharper Platform](Intro/ReSharperPlatform.md)
* [About This Guide](Intro/About.md)
* [Key Topics](Intro/KeyTopics.md)
* [Getting Help](Intro/GettingHelp.md)

## Part I - Extending the ReSharper Platform
* [Getting Started](Extensions/GettingStarted.md)
    * [Tools](Extensions/Tools.md)
    * [Internal mode](Extensions/InternalMode.md)
    * [Platform Versioning](Extensions/PlatformVersioning.md)
* [What's New?](Extensions/WhatsNew.md)
* [Declarative Extensions](Extensions/DeclarativeExtensions.md)
    * Live Templates
    * Structural Search and Replace Patterns
    * External Annotations
        * Attribute Reference
        * Annotator Tool
* [Compiled Extensions (Plugins)](Extensions/CompiledExtensions.md)
    * [Project Setup](Extensions/Plugins/ProjectSetup.md)
        * [Creating Projects](Extensions/Plugins/ProjectSetup/CreatingProjects.md)
        * [NuGet References](Extensions/Plugins/ProjectSetup/NuGetReferences.md)
        * [Initial Installation](Extensions/Plugins/ProjectSetup/InitialInstallation.md)
        * [Copy Plugin on Build](Extensions/Plugins/ProjectSetup/CopyOnBuild.md)
    * [Debugging](Extensions/Plugins/Debugging.md)
    * [Plugin Testing](Extensions/Plugins/Testing.md)
        * [Project Structure](Extensions/Plugins/Testing/ProjectStructure.md)
        * [Test Environment](Extensions/Plugins/Testing/TestEnvironment.md)
        * [Zones](Extensions/Plugins/Testing/Zones.md)
        * [Test Components](Extensions/Plugins/Testing/TestComponents.md)
        * [Gold Files](Extensions/Plugins/Testing/GoldFiles.md)
        * Base Classes
        * Attributes
        * Project References
        * [Temporary Settings](Extensions/Plugins/Testing/TemporarySettings.md)
        * [Combinatorial Settings](Extensions/Plugins/Testing/OptionsIterator.md)
        * Utility Classes
        * Reference
    * [Known Issues](Extensions/KnownIssues.md)
* [Deployment](Extensions/Packaging.md)
    * Installing the ReSharper Platform
        * [Host Identifiers](Extensions/Deployment/InstallProcess/HostIdentifiers.md)
        * [Dependencies](Extensions/Deployment/InstallProcess/Dependencies.md)
    * Packaging Details
    * Localisation
    * [Installing Locally](Extensions/Deployment/LocalInstallation.md)
        * [Experimental Instances](Extensions/Deployment/LocalInstallation/ExperimentalInstance.md)
        * [Package the Extension](Extensions/Deployment/LocalInstallation/Packaging.md)
        * [Install Local Copy](Extensions/Deployment/LocalInstallation/InstallCustomSource.md)
        * [Copy Plugin on Build](Extensions/Deployment/LocalInstallation/CopyOnBuild.md)
    * [Supporting ReSharper 8](Extensions/Packaging8.md)
* [Troubleshooting](Extensions/Troubleshooting.md)
* Hints and Tips
* Best Practices

## Part II - Architectural Overview
* [Overview](Architecture/Overview.md)
* Platform
* Project Model
* [PSI](Architecture/PSI.md)
* Features
* Products
    * Standalone
    * Deployment

## Part III - Platform
* [Logging](Platform/Logging.md)
    * [Advanced Configuration](Platform/Logging/AdvancedConfiguration.md)
    * Usage
    * Category Reference
* [Lifetime](Platform/Lifetime.md)
    * [Usage](Platform/Lifetime/Usage.md)
    * [Lifetime Management](Platform/Lifetime/LifetimeDefinition.md)
    * [Advanced Lifetime Management](Platform/Lifetime/Advanced.md)
    * [Component Lifetime](Platform/Lifetime/ComponentModel.md)
    * [Case Study](Platform/Lifetime/CaseStudy.md)
* DataFlow
    * IViewable
    * IProperty
    * ISignal
* [Component Model](Platform/ComponentModel.md)
    * [Containers, Parts and Catalogues](Platform/ComponentModel/ContainersPartsCatalogues.md)
    * [Visual Studio interfaces](Platform/VisualStudio/ComponentModel.md)
* [Zones](Platform/Zones.md)
    * [Implementation](Platform/Zones/Implementation.md)
    * [Usage](Platform/Zones/Usage.md)
    * [Features and Products](Platform/Zones/FeaturesProducts.md)
    * [Activation](Platform/Zones/Activation.md)
    * Licensing
    * Troubleshooting
    * [How to Define a Zone Marker](Platform/Zones/HowTo.md)
    * Reference
* DataContext
* [TextControl](Platform/TextControl.md)
* Documents
* Threading
* Locks
* Transactions
* IL Metadata
* Shell
    * Visual Studio integration
        * [Experimental Instances](Platform/VisualStudio/CustomHives.md)
    * Command line
    * [Tool windows](Platform/Shell/ToolWindows.md)
    * [Icons](Platform/Shell/Icons.md)
        * [Consuming Icons](Platform/Shell/Icons/ConsumingIcons.md)
        * [Creating Compiled Icons](Platform/Shell/Icons/CreatingCompiledIcons.md)
        * [Icon Types](Platform/Shell/Icons/IconTypes.md)
        * [Custom Icon Types](Platform/Shell/Icons/CustomIconTypes.md)
    * Tooltips
    * TreeView
    * Progress reporting
    * Popup menus
    * Extensions
    * Theming
        * [Themed Icons](Platform/Shell/Theming/Icons.md)
* [Settings](Platform/Settings.md)
    * Reading and writing
    * Defaults
    * Layers
    * File format
    * Util

## Part IV - Project Model
* Overview
* Change Manager
* MSBuild Integration
* Project Files

## Part V - PSI
* [Overview](Architecture/PSI.md)
* PSI Languages
    * [Per-language Components](PSI/PerLanguageComponents.md)
* Files and Modules
* [Syntax Trees](PSI/SyntaxTrees.md)
* [Navigating Syntax Trees](PSI/NavigatingSyntaxTrees.md)
    * [Manual Navigation](PSI/SyntaxTrees/ManualNavigation.md)
    * Helper methods
    * [Strongly Typed Navigation](PSI/SyntaxTrees/StronglyTypedNavigation.md)
    * Recursive navigation
    * Finding nodes
* Manipulating syntax trees
    * Transactions
* [Generating code](PSI/Generating/Code.md)
    * [Generating comments](PSI/Generating/Comments.md)
* [Declared Elements](PSI/DeclaredElements.md)
    * Declared Element Types
    * Declared Element Envoys
* [References](PSI/References.md)
    * Resolving references
    * [Reference providers](PSI/References/ReferenceProviders.md)
    * [Code Completion](PSI/References/CodeCompletion.md)
    * [References within elements](PSI/References/Substrings.md)
    * [Managing reference providers](PSI/References/IReferenceProvider.md)
    * [Path references](PSI/References/PathReferences.md)
    * Colour references
* [Type Systems](PSI/TypeSystem.md)
    * [Overview](PSI/TypeSystems2.md)
* Symbols and Symbol Tables
* Constant Evaluation
* Control Flow Graphs
* [Caching](PSI/Caching.md)
    * Persistent Caches
        * Symbol Cache
        * Word Index
    * Persistent Indexes
    * Transient Caches
* Searching
* Pointers
* External Annotations
    * [Testing](PSI/ExternalAnnotations/Testing.md)
* [PSI Icons](PSI/Icons.md)
* PSI Implementations Reference
    * C#
    * Visual Basic
    * JavaScript
    * TypeScript
    * JSON
    * HTML
    * CSS
    * Razor
    * ASP.net
    * [XML and DTD](PSI/Reference/Xml/Overview.md)
        * [Manipulating the Tree](PSI/Reference/Xml/Manipulation.md)
        * [Navigators](PSI/Reference/Xml/Navigators.md)
        * [Element Factories](PSI/Reference/Xml/ElementFactories.md)
        * [Utils](PSI/Reference/Xml/Utils.md)
        * [Tree Nodes](PSI/Reference/Xml/TreeNodes.md)
        * [Tree Nodes (DTD)](PSI/Reference/Xml/TreeNodesDTD.md)
    * Build scripts
    * XAML
    * XML Doc

## Part VI - Features
* Analysis
    * Highlights
    * Custom Highlights
    * Suppressing Highlights
    * [Daemons](Features/Analysis/Daemons.md)
    * Element problem analysers
    * Xml and Html analysers
    * Solution Wide Analysis
    * Error Stripe
    * Gutter Marks
    * Syntax Highlighting
    * Caches again
    * Value Analysis
    * Call Hierarchy
    * [Testing](Features/Analysis/Testing.md)
* [Actions](Features/Actions.md)
    * Menu items
    * [Quick fixes](Features/Actions/QuickFixes.md)
    * Context actions
    * [Bulk actions](Features/Actions/Bulk.md)
    * Overriding
    * Temporary editor actions
    * Action ID reference
* [Code Completion](Features/Completion.md)
    * Basic, smart and import
    * Dynamic completion
    * Double completion
    * Lookup items
* Contextual highlighting
    * Brace matching
    * string.Format style highlighting
    * Find usages in file style highlighting
* Popup docs
    * [Quick Doc](Features/QuickDoc/Overview.md)
        * [IQuickDocProvider](Features/QuickDoc/ImplementingProvider.md)
        * [IQuickDocPresenter](Features/QuickDoc/ImplementingPresenter.md)
        * [Helper classes](Features/QuickDoc/HelperClasses.md)
        * [Existing providers](Features/QuickDoc/ExistingProviders.md)
        * [Testing](Features/QuickDoc/Testing.md)
    * Parameter Info
* [Refactoring](Features/Refactoring.md)
    * Workflow
    * Integrating
    * Renaming
    * Unused references
* [Generate Menu](Features/GenerateMenu.md)
* [Navigation](Features/Navigation.md)
    * Go to everything
    * Go to related
    * Go to next/prev
    * Contextual - find exits
    * Navigate from here
    * Occurrences
* [Options Pages](Features/OptionsPages.md)
    * Search in Options Pages
* Tools
    * [Code cleanup](Features/Tools/CodeCleanup.md)
    * Code structure
    * Todo items
    * Bookmarks
    * Assembly explorer
* Naming
    * Suggestions
    * Policy
* Editing
    * Formatting
    * Typing assist
    * Complete statement
    * Extend selection
    * Rearrange code
    * Comments
* Structural Search and Replace
    * C# vs HTML and CSS
    * Programmatic usage
* [Live Templates](Features/LiveTemplates.md)
    * [Macros](Features/LiveTemplates/Macros.md)
    * [Scopes](Features/LiveTemplates/Scopes.md)
    * Categories
    * Quick lists
    * Hotspots
    * Programmatic usage
* [Unit Test](Features/UnitTest.md)
    * Test discovery
    * Test execution
    * Non-CLR tests
* Diagramming
    * Architecture diagrams
* External sources
    * PDB symbols
    * Decompiling
    * Source servers
* Asp.net MVC
* Internationalisation
* [Working with XML-like files](Features/XmlLike.md)

## Part VII - Products
* dotPeek
* [InspectCode](Products/InspectCode.md)
* Custom products

## Part VIII - Custom languages
* [Overview](CustomLanguages/Overview.md)
* [Registering a Custom Language](CustomLanguages/Registration.md)
    * [Project File Type](CustomLanguages/Registration/ProjectFileType.md)
    * [PSI Language Definition](CustomLanguages/Registration/PsiLanguageDefinition.md)
    * [Per-language Components](CustomLanguages/Registration/PerLanguageComponents.md)
    * [Project File Language Service](CustomLanguages/Registration/ProjectFileLanguageService.md)
    * [PSI Language Service](CustomLanguages/Registration/PsiLanguageService.md)
    * [Testing](CustomLanguages/Registration/Testing.md)
* Secondary PSI
* [Injected PSI](CustomLanguages/InjectedPsi.md)
* [Building the PSI tree](CustomLanguages/BuildingPsiTree.md)
    * [Node Types](CustomLanguages/Parsing/NodeTypes.md)
        * [Token Node Types](CustomLanguages/Parsing/NodeTypes/TokenNodeTypes.md)
        * [Composite Node Types](CustomLanguages/Parsing/NodeTypes/CompositeNodeTypes.md)
        * [Creating Node Types](CustomLanguages/Parsing/NodeTypes/CreatingNodeTypes.md)
    * [Lexing](CustomLanguages/Parsing/Lexing.md)
        * [The Tao of Writing a Lexer](CustomLanguages/Parsing/Lexing/Tao.md)
        * [Lexer Factories](CustomLanguages/Parsing/Lexing/LexerFactories.md)
        * [Text Buffers](CustomLanguages/Parsing/Lexing/TextBuffers.md)
        * [Implementing a Lexer](CustomLanguages/Parsing/Lexing/ImplementingLexers.md)
        * [Using CsLex](CustomLanguages/Parsing/Lexing/CsLex.md)
        * [Lexer Utility Methods](CustomLanguages/Parsing/Lexing/LexerUtil.md)
        * [Lexer Utility Classes](CustomLanguages/Parsing/Lexing/UtilityClasses.md)
        * [Filtering Lexers](CustomLanguages/Parsing/Lexing/FilteringLexers.md)
        * [Caching Lexers](CustomLanguages/Parsing/Lexing/CachingLexers.md)
        * [Testing](CustomLanguages/Parsing/Lexing/Testing.md)
    * [Parsing](CustomLanguages/Parsing/Parsing.md)
        * PsiGen and .psi Files
        * TreeBuilder
        * [Testing](CustomLanguages/Parsing/Parsing/Testing.md)
    * Extending Existing Languages
    * Incremental Parsing
* Creating Declared Elements
* Language Specific Features
    * Disabling Intellisense
