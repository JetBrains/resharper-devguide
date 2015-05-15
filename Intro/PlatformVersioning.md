---
---

# Platform Versioning

ReSharper does **not** maintain backwards binary compatibility between major or minor releases. For example, plugins written for ReSharper 8.2 do not work with ReSharper 8.1, and vice versa. The same is true for ReSharper 7.x and 8.x. Plugins need to be re-compiled between major and minor versions. However, maintenance releases **are** binary compatible - maintenance releases, such as 8.2.2 or 8.2.3, are intended to fix critical bugs, and do not usually introduce new features that would change APIs.

The reasons for this are fairly straightforward:

* **Evolving the API**. ReSharper needs to evolve its own API in order to provide new functionality, or to implement support for new language features.
* **Changes to supported languages**. If a supported language changes, introducing a new language feature (e.g. C# 6 introduces the `?.` null propagating operator), then new functionality is required to support this, and this usually introduces breaking changes. More importantly, the abstract syntax tree for the language changes - nodes in the tree can get new properties and children, or existing children need to be reworked to allow for new usages. Put shortly, changes to a language require breaking changes in the AST/API.

## Versioning the ReSharper Platform

With the move to the shared binary distribution of the ReSharper Platform, maintaining a track of the version of the installed products can be difficult - the initial release of the Platform includes ReSharper 9.0, dotCover 3.0, dotPeek 1.3, etc. In order to make this easier, the version of the ReSharper Platform is known as a "Wave". The initial version released with ReSharper 9.0 is known as "Wave 01".

Previous versions of the .net tools were intended to be built and distributed on their own timescales. Each product shared common code via source, and could be installed separately, and updated at any time. In practice, however, because dotCover and dotTrace need to integrate with ReSharper's test runner, a new version of ReSharper would require a simultaneous release of other .net tools, removing a large reason for the independence of the tools.

The ReSharper Platform is a shared binary platform for all of the .net tools, so it is an explicit dependency. When the Platform updates, all Products must also be updated. Nothing has changed for the end user here. It is now a more explicit and manageable dependency. Since all Products must be updated at the same time, the combination of the versions is known as a "Wave". The desire is to provide new "Wave" releases on a predictable schedule.

Products and extensions need to take a dependency on a particular version of a Wave. They will work with that version of a Wave, but not with any other version - when a new Wave is released, the extension needs to be updated to work with it. And of course, a Product and extension can be updated within a Wave, allowing for maintenance and other backwards compatible releases.
