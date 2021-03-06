---
title: CSS-Development
tags: [ontowiki]
sidebar: ontowiki_sidebar
permalink: /CSS-Development.html
editme_path: ontowiki/CSS-Development.md
---
OntoWiki has its own framework for HTML/CSS which is been used for the GUI (together with Javascript components).

## CSS Development

OntoWiki do not support Internet Explorer (IE) 6 anymore (so we can now use chained classes and better selectors for example), but we still have some relicts from that time in our CSS framework. Addionally we added elements and style options for different things during the past which could be merged to generalized concepts now. This means we can develop the framework step by step to a more elegant and usable solution.

### Stylesheets

All themes should be use following stylesheet files from the styles folder:

- **default.css** - home of the OntoWiki's CSS framework
- **default.dev.css (since 0.8)** - testing new elements/options or quick&dirty adoptions
- **deprecated.dev.css (since 0.9.5)** - defines a red outline with 1px width for all definitions from `default.css` which are marked as deprecated (`outline:solid 1px red;`), the `outline` attribute is still not supported by IE7 but we're talking about developing environments :)
- **legacy.css (after 0.9.5)** - contains removed definitions from `default.css` which are marked as deprecated for at least one OntoWiki release
- **legacy.dev.css (after 0.9.5)** - should be contain definitions from `deprecated.dev.css` after they have been there for one OntoWiki release, the outline should be 2px width to distinguish deprecated and legacy elements

All `\*.dev.css` files are only loaded with activated debug mode (it's recommended for developers to activate the debug mode). Definitions from `\*.dev.css` overwrite/extend definitions from default.css (and later from legacy.css), because they are loaded after the default styles.

Please comment in `deprecated.dev.css` and `legacy.css` what elements should be used instead or how options have to be used properly. A recent example is:

```
/**
 * Inherited Minibuttons
 * @deprecated 0.9.5
 *
 * - ul.minibutton and li.minibutton is deprecated 0.9.5 for simplification of the default styles
 * - please use directly span.minibutton or a.minibutton
 */
ul.minibutton li a,
li.minibutton a
{
    outline:solid 1px red !important;

}
```

## CSS Classes

see [http://ontowiki.net/Projects/OntoWiki/DefaultStylesheet](http://ontowiki.net/Projects/OntoWiki/DefaultStylesheet) **needs new link**
