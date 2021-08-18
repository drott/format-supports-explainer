# src: descriptor `format(…supports…)` syntax for client side font selection

## Authors

* Dominik Röttsches, [@drott](drott@chromium.org)

## Introduction ##

With the introduction of variable fonts, the CSS WG
[started to discuss in issue 633](https://github.com/w3c/csswg-drafts/issues/633)
how authors can make use of the feature in browsers, while support in user
agents was still limited. There was a desire for having a fallback mechanism
based in `@font-face` `src:` descriptors which allows the UA to select the font
resource that is most advanced, based on the technologies it supports. This
allows authors to use variable fonts where supported by the UA, falling back to a
static font if needed.

Before that, the `src:` descriptors `format()` specifier was used to distinguish
the font resource's format itself ("opentype", "svg"...) and its used
compression mechanism, see
[CSS Fonts Level 3](https://drafts.csswg.org/css-fonts-3/#src-desc). Similarly
here, the intention was to have UAs select the type of font resource that they
support. If an author lists a highly compressed `src: url() format("woff2")`
resource first, a compatible UA chose this one first, and allow for faster
downloads. While a UA without WOFF2 support gracefully falls back to lower
compression efficiency or an uncompressed font.

In addition to variable fonts, with growing color font support and to recognize
that use of font technology features are part of the contents of the font file,
while the format itself can stay the same, the syntax for the `format()` part of
the `src:` was extended to have a `supports` extension, which allows a finer
grained description of the required capabilities to use this font resource entry
of the `src:` descriptor.

## Proposed Syntax

See
[4.3.1 Parsing the src descriptor of the CSS Fonts Level 4 spec](https://drafts.csswg.org/css-fonts-4/#font-face-src-parsing)

## Non-Goals

This proposal is not-intended as a server-side content negotiation solution. In
many cases, third-party font providers currently choose based on User Agent
which resources they deliver to clients at the time of the request to the
included CSS. This is a different content negotiation mechanism than what is
discussed in this proposal.

## Goals & Use Cases

### Optimal choice of supported font technology

The use case can be described as follows:

> As a an author, I want to be able to select the most advanced font resource
> that is supported by the UA in order to provide the best presentation of my
> content.

#### Example

```
@font-face {
  font-family: ColorFontIfPossible;
  src: url(//fonts/mycolorfont.woff2) format(woff2 supports COLRv1),
       url(//fonts/mycolorfont_fallback_colrv0.woff2) format(woff2 supports COLRv0),
       url(//fonts/mycolorfont_fallback_glyf.woff2) format(woff2);
}
```

A compatible UA that supports `COLRv1` fonts select the first resource and can
show vector color glyphs with gradients and compositing effects. If the UA does
not support `COLRv1`, but supports `COLRv0` the second resource is loaded, still
supporting layered color glyphs with unicolor-layers. Lastly, if there is no
support for those two formats, the UA selects the contour font resource without
color.

### Detectability

The use case can be described as follows:

> As an author, I want to know programmatically in my script code what level of font support is available.

This need is in line with the TAG design principles, which recommend
[detectability of a feature](https://w3ctag.github.io/design-principles/#feature-detect).

The wording of the specification currently reads:

> If a component value is parsed correctly and is of a format and font
> technology that the UA supports, add it to the list of supported sources. If
> parsing a component value results in a parsing error or its format or
> technology are unsupported, do not add it to the list of supported sources.

Using this rule, availability of a particular font technology can be
programmatically tested for by evaluating a `@font-face` rule and accessing its
result `src:` descriptor value. Without downloading any font resources,
knowledge about technology support can be retrieved.

Direct testing of font capabilities is not possible through the `CSS.supports()`
syntax as there is no reliable mapping between font technologies and CSS
properties and their values.

Testing of font capabilities is possible through probing for rendered pixels on
a 2D canvas and testing for RGB color values, as done in @RoelN's
[Chromacheck](https://pixelambacht.nl/chromacheck/) tool. This is a wasteful
approach from a resource point of view, requiring canvas resources for something
that can be returned by the UA and detected more efficienlty.

#### Example

```HTML
<style>
  @font-face {
    font-family: a;
    src: url(/FEATURETESTVAR) format(woff2 supports variations);
  }
</style>
<script>
  window.onload = () => {
    var variations_supported = document.styleSheets[0].cssRules[0].cssText.includes("FEATURETESTVAR")
        ? "Variations supported." : "Variations NOT supported.")
    console.log("Variations supported: " + variations);
  }
</script>
```
