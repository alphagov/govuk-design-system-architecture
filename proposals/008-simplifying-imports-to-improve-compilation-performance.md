# Simplifying imports within GOV.UK Frontend to improve compilation performance

**Closing date:** 1 May 2020

**Status:** Accepted

## Summary

**This is an iteration of [proposal #7](https://github.com/alphagov/govuk-design-system-architecture/pull/21), based on the feedback we received from that proposal.**

We’re making changes to the way that the base layers (settings, tools and helpers) are imported within different parts of GOV.UK Frontend’s Sass, to reduce the time it takes to compile to CSS.

Most of you will not be affected by these changes, and if you import "node_modules/govuk-frontend/govuk/all" you will not need to do anything to benefit from these performance improvements.

It will have the most impact on you if you import only specific parts of GOV.UK Frontend.

## Problem

It can take a long time to compile GOV.UK Frontend’s Sass to CSS. This is because many of the individual files within GOV.UK Frontend import their own dependencies, even if they would usually have already been imported. We originally did this so that you could import specific bits of GOV.UK Frontend without having to separately import their dependencies.

Reducing the number of imports within GOV.UK Frontend has been shown to significantly improve the compilation time, as demonstrated in [#1671 Slow compilation of SCSS files in Rails apps](https://github.com/alphagov/govuk-frontend/issues/1671).

This should particularly improve things for you if you use Sprockets to package your assets, although we expect these changes to improve the compilation time for everyone.

## Proposal

We will implement changes in two stages.

1. We can make changes to the imports within components without it being a breaking change, so we’ll do this as part of a v3.x release.
2. We’ll make the breaking changes to the core, objects and overrides layers in the v4.0 release.

### Changes in GOV.UK Frontend v3.x

We’ll add a new file called `_index.scss` to each component, which contains the component’s styles but does not import the settings, helpers and tools layers.

We’ll update the existing _[component].scss files to import the settings, helpers and tools layers and import _index.scss. This will preserve the existing behaviour, and continue to provide a single entry point for individual components that can be used in contexts such as React.

We’ll update  `components/_all.scss` to include every component’s `_index.scss` file, avoiding the unnecessary imports of the settings, helpers and tools layers within each component.

You can still import `/govuk-frontend/govuk/all`, and if you’re doing this you will not need to make any changes to benefit from these improvements.

You can still import components individually (for example `govuk-frontend/govuk/components/button/button`), but you should update your code to import the index file for each component (instead of the [component].scss) if you want to benefit from the performance improvements:

```scss
@import "govuk/settings/all";
@import "govuk/tools/all";
@import "govuk/helpers/all";

@import "govuk/components/accordion/index";
@import "govuk/components/button/index";
```

In preparation for GOV.UK Frontend v4.0, we’ll add deprecation warnings if you import files from the core and overrides layers without first importing the settings, helpers and tools layers. 

### Changes in GOV.UK Frontend v4.0

We’ll move the template styles from the core/_template.scss to objects/_template.scss, so that we treat it as an object rather than core styling.

We’ll remove the imports of the settings, helpers and tools layers from files in core and overrides. 

If you selectively import individual files from core or overrides, you’ll need to import the settings, tools and helpers layers first:

```scss
@import "govuk/settings/all";
@import "govuk/tools/all";
@import "govuk/helpers/all";

@import "core/typography";
```

We’ll continue to import the settings, helpers and tools layers from files within the objects layer, so that they can act as single entry points that can be used in contexts such as React.
