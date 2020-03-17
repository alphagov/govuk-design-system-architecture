# Remove the imports of settings, tools and helpers layers from components

**Closing date:** 27 March 2020

**Status:** Proposed

## Summary

We plan to remove the imports of the settings, helpers and tools layers from all components in GOV.UK Frontend version 4.0.

We’re doing this to reduce the time it takes to compile GOV.UK Frontend’s Sass to CSS, especially if you’re using the Ruby Sass compiler.

## Problem

If you're using the Ruby Sass compiler to compile Sass to CSS, it can take a long time because GOV.UK Frontend has a lot of imports.

Although Ruby Sass is deprecated, teams (including GOV.UK) are stuck using it for the foreseeable future.

If we make these changes, we'll also improve the compile time in other Sass compilers. Approximately, we expect to see these improvements:

| Compiler  | Compile time now | Compile time after proposed changes | Improvement |
|-----------|------------------|-------------------------------------|-------------|
| Ruby Sass | 30s              | 7.8s                                | 74%         |
| LibSass   | 0.2s             | 0.18s                               | 10%         |
| Dart Sass | 0.5s             | 0.25s                               | 50%         |

For more context, see [#1671 Slow compilation of SCSS files in Rails apps](https://github.com/alphagov/govuk-frontend/issues/1671).

## Proposal

We propose removing the following imports of the settings, helpers and tools layers from each component:

```scss
@import "../../settings/all";
@import "../../tools/all";
@import "../../helpers/all";
```

This will have no effect if you currently import `govuk/all`, as all of the layers are already imported in order.

If you selectively import one or more components, you would need to make sure you import the settings, tools and helpers layers first:

```scss
@import "govuk/settings/all";
@import "govuk/tools/all";
@import "govuk/helpers/all";

@import "govuk/components/accordion/accordion";
@import "govuk/components/button/button";
```

As this changes the way that some users would import GOV.UK Frontend, this would be a breaking change, which we'd introduce in GOV.UK Frontend version 4.0.
