# Zones

Zones provide dependency management for features. Code can declare what Zones it requires and exports, enabling tooling and ensure correct layering, packaging and deployment. Features, implemented as Zones, can be independently licensed and disabled. Products can be created by combining Features, and multiple products can be efficiently installed by sharing overlapping Features.

## Background

Zones were introduced with ReSharper 9.0 to enable a shared binary distribution of products built on the ReSharper platform.

Previously, ReSharper, dotCover, dotTrace, etc. shared a common codebase at the source level. This allowed for standalone installations of the products, even allowing for different versions of the common codebase to be installed together. However, this approach causes increased resource usage when multiple products are installed in Visual Studio. The de-coupling also doesn't solve the problem of co-ordinated release schedules for integration points (e.g. dotCover needs to be updated to integrate with a new version of ReSharper's test runner).

Clearly, a better strategy is to share a binary implementation of the common codebase. However, this introduces complexity when installing multiple products. Each product requires a different feature set, and it is the intersection of all features from all products that should be installed.

One approach would be to install the superset of all referenced assemblies, but this can force assembly packaging decisions based on product usage, rather than for architectural reasons. For example, dotCover might require NUnit test discovery, but not the analysis and highlights for test attribute usage. Or the NUnit feature supports both C# and Visual Basic, but the user has only purchased support for C# in ReSharper. These scenarios might force the NUnit support to ship in three (or more!) separate assemblies, increasing maintainability and runtime overheads (remember, assemblies are relatively expensive).

Zones provide an abstraction to manage this complexity by integrating with the Component Model and only enabling components that are parts of installed and licensed Zones. Any component that is not part of an installed, active Zone is not available to the application. This decouples the product from the physical packaging of components into assemblies, and allows decisions on shipping and packaging to be made for architectural and layering reasons, rather than trying to satisfy combinations of product boundaries.
