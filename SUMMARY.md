# Summary

## Part I
* [Work in progress](wip.md)
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
    * Packaging and Distribution
* [How do I?](HowDoI.md)
* Plugins and SDK
    * Debugging
    * [Known Issues](Intro/KnownIssues.md)
    * Plugin testing
        * Base classes
        * [Temporary Settings](Plugins/TemporarySettings.md)
        * Quick links
        * Testing code completion
        * Testing unit test support
        * Testing new language support
* [Glossary](Intro/Glossary.md)

## Part II - Architectural Overview
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
* Component Model
* DataContext
* TextControl
* Documents
* Threading
* Locks
* Transactions
* IL Metadata
* Shell
    * Visual Studio integration
    * Command line
    * Tool windows
    * Icons
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
* Settings
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
* Generating code
* Semantic models
* Path and color references
* Control flow
* Caches
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
    * Daemons
    * Element problem analysers
    * Xml and Html analysers
    * Solution Wide Analysis
    * Error Stripe
    * Syntax Highlighting
    * Caches again
    * Value analysis
    * Call hierarchy
* Actions
    * Menu items
    * Quick fixes
    * Context actions
    * Bulk actions
    * Overriding
    * Temporary editor actions
    * Action ID reference
* Code Completion
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
* Refactoring
    * Workflow
    * Integrating
    * Renaming
    * Unused references
* Generate This
* Navigation
    * Go to everything
    * Go to related
    * Go to next/prev
    * Contextual - find exits
    * Navigate from here
    * Occurrences
* Tools
    * Code cleanup
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
* Live Templates
    * Macros
    * Scopes and categories
    * Hotspots
    * Programmatic usage
* Unit Test
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
* Disabling language support
* Project model language support
* PSI language support
* Secondary PSI
* Injected PSI
* Lexing and parsing
* psi files
* PsiGen
