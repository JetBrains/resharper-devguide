# Implementation

When the application first starts up, the Component Model creates a collection of all of the types in the application that are marked with an attribute deriving from `PartAttribute` (known as "parts"). The shell and solution containers are created by filtering this collection based on the type of the part attribute, i.e. `[ShellComponent]` or `[SolutionComponent]`. Zones are implemented by a similar filter.

This filter is given a list of available zones for the application, based on installed products, the host environment and user preferences (e.g. whether the zone has been disabled). When evaluating a component, the filter will look to see what zones it belongs to. If all of that component's zones are available, it is allowed through the filter. If any zones are not available, the component is filtered out, and does not get loaded by the application.

For example, if one of the initial active zones when the application starts is `IProjectModelZone`, then any component marked as belonging to that zone will be loaded. In contrast, if a component is marked with both `IProjectModelZone` and `IUnitTestingZone`, the component will not be loaded, because the `IUnitTestingZone` zone definition isn't a currently active zone. This allows features to be shipped and installed, but not available.

Similarly, zones can be licensed, so a zone may be installed, but unavailable due to an expired (or trial) license.

Furthermore, zones are hierarchical, and can have dependencies. For example, the "ILanguageCSharpZone" depends on the "IClrPsiLanguageZone" and all of its dependencies:

```
ILanguageCSharpZone
- IClrPsiLanguageZone
  - IPsiAssemblyFileLoaderImplZone
  - IPsiLanguageZone
    - PsiFeaturesImplZone
      - ITextControlsZone
      - IProjectModelZone
      - IDocumentModelZone
```

When code is marked as belonging to the "ILanguageCSharpZone", it is automatically marked as also belonging to all of that zone's dependencies. All of these zones must be active before any of these components are added to the Component Model.

