# New version of font

**Date:** 2019-07-29

**Status:** Accepted

## Context

There were a [number of issues with "NTA"](https://github.com/alphagov/govuk-frontend/issues/1012), the previous version of the font:

1. The file size for all versions of the font was larger than it needed to be.
2. A separate font was required for tabular numbers.
3. The glyphs sat too high in the fonts bounding box (baseline) which led to the need for adjustments in CSS.
4. Hinting requirement for oldIE (EOT) increased the file size even further.

## Decision

- Replace the "NTA" font with a new version, called "GDS Transport", which fixes the above issues.
- Specifically, stop serving Transport font to IE8 due to a [large file required for hinting](https://designnotes.blog.gov.uk/2019/07/29/weve-updated-the-gov-uk-colours-and-font/). The font now [falls back to Arial](https://github.com/alphagov/govuk-frontend/blob/master/CHANGELOG.md#gds-transport-now-falls-back-to-arial-in-internet-explorer-8-ie8) for IE8.

## Consequences
- Font files are now smaller: WOFF2 files are ~33kb and ~31kb, compared with ~66kb and ~54kb of the previous version of the font 🎉
- The vertical alignment of the font has been corrected. This removes the need for manually adjusting the baseline of the font.
- Tabular numbers are now part of the [`govuk-font` mixin](http://govuk-frontend-review.herokuapp.com/docs/#helpers-mixin-govuk-font), instead of a separate font family.
- The previous version of the font is now deprecated. Service teams using the legacy code bases [can still use the previous version in compatibility mode](https://github.com/alphagov/govuk-frontend/blob/master/docs/installation/compatibility.md).

## Advantages

- Improved page performance due to smaller font files.
- No need to adjust the font baseline manually. This means simpler code and less design / development work.
-  With the baseline adjustments removed, the font can be swapped for a different one such as Arial without manually having to remove the baseline adjustments. An example would be if [the service is not on gov.uk](https://www.gov.uk/service-manual/design/making-your-service-look-like-govuk#if-your-service-isnt-on-govuk).
- Simpler documentation for using tabular numbers which are now part of the `govuk-font` mixin. Using tabular numbers no longer requires loading in a separate font, leading to performance gains.
- IE8 will not download the font files but falls back to a default font, leading to performance gains.

## Disadvantages

- Service teams might need to make adjustments in their custom components which contain corrections to the baseline of the font.
