# Summary

## Prefix
* [Style guide](StyleGuide.md)
* [Work in progress](wip.md)

## Part I
* Getting Started
    * Tools
    * Internal mode
    * Platform Versioning
* Extensions Overview
    * Live Templates
    * Structural Search and Replace Patterns
    * External Annotations
        * Attribute Reference
        * Annotator Tool
    * Plugins and SDK
    * [Packaging and Distribution](Extensions/Packaging.md)
    * Troubleshooting
* [How do I?](HowDoI.md)
* Plugins and SDK
    * Debugging
    * [Known Issues](Intro/KnownIssues.md)
    * [Plugin testing](Plugins/Testing.md)
        * Base classes
        * [Temporary Settings](Plugins/TemporarySettings.md)
        * Quick links
        * Testing code completion
        * Testing unit test support
        * Testing new language support
* [Glossary](Intro/Glossary.md)

## Part II - Architectural Overview
* [Overview](Architecture/Overview.md)
* Platform
* Project Model
* PSI
* Features

## Part III - Platform
* Logging
* DataFlow
* [Lifetime](Platform/Lifetime.md)
    * [Usage](Platform/Lifetime/Usage.md)
    * [Lifetime Management](Platform/Lifetime/LifetimeDefinition.md)
    * [Advanced Lifetime Management](Platform/Lifetime/Advanced.md)
    * [Component Lifetime](Platform/Lifetime/ComponentModel.md)
    * [Case Study](Platform/Lifetime/CaseStudy.md)
* [Component Model](Platform/ComponentModel.md)
    * [Visual Studio interfaces](Platform/VisualStudio/ComponentModel.md)
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
    * Tooltips
    * TreeView
    * Progress reporting
    * Popup menus
    * Extensions
    * Theming
* Project Model
    * Change management
    * MSBuild integration
    * Project files
* [Settings](Platform/Settings.md)
    * Reading and writing
    * Defaults
    * Layers
    * File format
    * Util

## Part IV - PSI
* Syntax trees
* [Navigating syntax trees](PSI/NavigatingSyntaxTrees.md)
    * Helper methods
    * [Strongly typed navigation](PSI/SyntaxTrees/StronglyTypedNavigation.md)
    * Recursive navigation
    * Finding nodes
* Manipulating syntax trees
* References
    * Resolving references
    * [Reference providers](PSI/References/ReferenceProviders.md)
    * [Code Completion](PSI/References/CodeCompletion.md)
    * [References within elements](PSI/References/Substrings.md)
    * [Managing reference providers](PSI/References/IReferenceProvider.md)
    * [Path references](PSI/References/PathReferences.md)
    * Colour references
* [Generating code](PSI/Generating/Code.md)
    * [Generating comments](PSI/Generating/Comments.md)
* Semantic models
    * [Type system](PSI/TypeSystem.md)
* Control flow
* [Caches](PSI/Caches.md)
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
    * XML
    * Build scripts
    * XAML
    * XML Doc

## Part V - Features
* Analysis
    * Highlights
    * Custom highlights
    * Suppressing highlights
    * [Daemons](Features/Analysis/Daemons.md)
    * Element problem analysers
    * Xml and Html analysers
    * Solution Wide Analysis
    * Error Stripe
    * Syntax Highlighting
    * Caches again
    * Value analysis
    * Call hierarchy
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
    * Progammatic usage
* [Live Templates](Features/LiveTemplates/Overview.md)
    * [Macros](Features/LiveTemplates/Macros.md)
    * Scopes and categories
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

## Part VI - Custom languages
* [Overview](CustomLanguages/Overview.md)
* Disabling language support
* Project model language support
* PSI language support
* Secondary PSI
* [Injected PSI](CustomLanguages/InjectedPsi.md)
* Lexing and parsing
* psi files
* PsiGen
