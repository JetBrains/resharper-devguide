#ReSharper DevGuide

You can view this book at http://citizenmatt.github.io/resharper-devguide. It's an experiment in:

* [GitBook](http://www.gitbook.io/)
* GitHub pages
* Restructuring the ReSharper dev guide

> **Note** It is *not* a replacement for the current [ReSharper devguide](http://confluence.jetbrains.com/display/NETCOM/ReSharper+8+Plugin+Development).

Consider this a playground to try out a new structure to the devguide without causing disruption to the existing site.

I want to see:

* What would a complete table of contents for ReSharper extensibility look like?
* How would I document the PSI? Language support?
* How do you best document the fundamentals of plugin dev, such as Component Model and Lifetime? Get people up and running without having to learn the whole architecture. I.e. balance tutorials and reference.
* How would I provide both an introduction to a topic and also more detailed docs (think code completion, intro and basic, smart, import, double, dynamic, matchers, lookup items, wrapping look up items...). Possibly answered by the last point

In other words, an experiment in starting the devguide from scratch.

Issues and Pull Requests can be raised on [GitHub](https://github.com/citizenmatt/resharper-devguide).

## Conventions ##

In the [chapter list](summary.md), I'm assuming that the top level node will have an introduction to the topic, e.g. Extensions would discuss the different types of extensions and provide links to more information, or code completion would show bare bones example of how to add items into the list. The next level down would provide a set of pages that provide more detail, e.g. extension packaging, double completion, overriding completion items, etc.
