[//]: # (title: How to Define a Zone Marker)

When creating an extension, it is important to get the right dependencies for a `ZoneMarker`. Zones are used to be able to correctly load a product in the correct host. If the zone marker is incorrect, either the components in that namespace are not loaded and the extension is not available, or the extension is loaded when it shouldn't be, and can cause errors in hosts as expected dependencies are not available.

ReSharper ships with diagnostics to help create accurate zone markers. **Unfortunately**, the diagnostics currently only work when all zones are defined in source (i.e. when you're building the ReSharper solution). When building an extension that references the ReSharper Platform via assembly references (i.e. through the SDK), the diagnostics **do not work**. An updated version of the diagnostics is being worked on, and will be available as a separate, SDK tools extension shortly.

(The diagnostics check the zone of each component used in the project, e.g. type usages in component constructors. If the zone isn't explicitly or implicitly required by the current zone marker, the type is highlighted, and a quick fix is available to add the required zone to the zone marker.)

Without diagnostics, it is very hard to work out what zones should be required. If you wish to use a component in your code, you need to discover which zone definitions it requires, and include those in your zone marker. This is done by finding the `ZoneMarker` class, and requiring the same zone definitions. This can be done by hand, by finding zone marker classes that live in the same namespace or higher as those used in your `using` statements, but this is tedious and prone to error.

A temporary solution is to use an [empty zone marker](Zones_Usage.md#empty-zone-marker). This is usually reserved for infrastructure code that has no real dependencies and should always run. However, as calculating the proper requirements is currently prohibitive, it should be used (TEMPORARILY) for extensions.

 >  This guidance is TEMPORARY! It is STRENUOUSLY RECOMMENDED to provide accurate and minimal zone information, in order that your extension is correctly loaded into the appropriate hosts, and does not cause issues for other hosts.
> 
> Without proper dependency information, extensions CAN and WILL FAIL. E.g. an extension intended to extend ReSharper will cause problems when run in a host with only dotCover installed.
>
> This guidance will be updated when the appropriate tooling is available to help calculate the correct requirements for zone markers.