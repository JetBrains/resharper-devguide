# Contributing

In order to build the site locally, you will need Ruby installed (the site is based on [Jekyll](http://jekyllrb.com)). The steps to get the site built are:

1. Install Ruby
2. Install Bundler - `gem install bundler`
3. Clone this repo
4. Initialise and clone the sub-modules - `git submodule init` and `git submodule update`
5. Download dependencies (gems) - `rake bootstrap`
6. Build and preview the site with `rake preview`

This starts Jekyll running the site at `http://localhost:4001/help/resharper/sdk/`

## Notes on style

### Titles

* Title case for page headings, both in the page and in the `_SUMMARY.md` file.
* Sentence case for sub headings in pages

### Terms

The following terms are used to refer to standard well known features in the product. They are not normally capitalised, unless otherwise shown.

* Alt+Enter, mono space (should wrap in `<kbd>Alt</kbd>+<kbd>Enter</kbd>`?)
* Context action
* Quick-fix
* Code cleanup
* plugin

### Code examples

* Indent of 2. Be conservative with space
* Long lines will scroll horizontally, so don't wrap unnecessarily. However, don't add really long lines either.
* Allow blank lines for clarity
* Comment if necessary, but not for everything
