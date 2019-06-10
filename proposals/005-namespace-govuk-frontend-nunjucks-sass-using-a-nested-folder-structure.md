# Namespace GOV.UK Frontend Nunjucks and Sass using a nested folder structure

**Closing date:** 2019-06-12

**Status:** proposed

## Summary

We want to namespace `govuk-frontend` by changing the folder structure for the `govuk-frontend` npm package.

We are doing this to make it easier for users to import things, in a way that is explicit and avoids conflicts in their application.

This would be a breaking change in the 3.0.0 release.

## Problem

To import things from GOV.UK Frontend, your application needs to know where to find those things in the filesystem.

You can do this by providing the full path to the file, for example

```javascript
{% from "node_modules/govuk-frontend/components/accordion/macro.njk" import govukAccordion %}
```

You can also tell your application which 'include paths' to look in to find things, which allows you to simplify the import path.

In GOV.UK Frontend, you add the following paths to your 'include paths':

- `node_modules/govuk-frontend/`
- `node_modules/govuk-frontend/components/`

You can then import things like this:

```javascript
{% from "accordion/macro.njk" import govukAccordion %}
```

However, there are 2 issues with this approach at the minute.

1. It's no longer clear where the thing is coming from. Those new to the codebase may not realise that it's being imported from an external dependency, because it's not explicit.

2. If you create your own accordion component, your application will not know which one to import (or it'll just try and import the first one it finds) as there will be two paths that both match.

There are 2 ways to get around these issues at the moment:

1. You could add the `node_modules` path to your 'include paths'.

   This then allows you to include things referencing govuk-frontend, like this:

   ```javascript
   {% from "govuk-frontend/components/accordion/macro.njk" import govukAccordion %}
   ```

   However, this is not practical since your application has to search all of the packages in the `node_modules` folder to try and find the thing, which can be very slow.

2. Using a custom namespacing in the your codebase, for example:

   ```javascript
   {% from "app-accordion/macro.njk" import appAccordion %}
   ```

## Proposal

Create a new folder called `govuk/` and move all the GOV.UK Frontend source files into it.

```bash
.
├── README.md
├── govuk
│   ├── all-ie8.scss
│   ├── all.js
│   ├── all.scss
│   ├── assets
│   ├── common.js
│   ├── components
│   ├── core
│   ├── helpers
│   ├── objects
│   ├── overrides
│   ├── settings
│   ├── template.njk
│   ├── tools
│   ├── utilities
│   └── vendor
├── govuk-prototype-kit.config.json
└── package.json
```

Then you could add the following path to your 'include paths':

- `node_modules/govuk-frontend/`

You could then import components like this:

**Nunjucks:**
```javascript
{% from "govuk/components/accordion/macro.njk" import govukAccordion %}
```

**Sass:**

```scss
@import "govuk/components/accordion/accordion"
```

**JavaScript:**

```javascript
import Accordion from "govuk/components/accordion/accordion"
```

Because we can specify [the primary entry point in the package.json](https://docs.npmjs.com/files/package.json#main), you could also import components like this:

```javascript
import { Accordion } from "govuk-frontend"
```

You can see the [proposed package folder structure](https://github.com/alphagov/govuk-frontend/tree/fb0476423e28eb0410c6810d4ff6ef4ef17d2df2) on GitHub.

If you'd like to try this out in your application, you can install a pre-release:

```bash
npm install --save alphagov/govuk-frontend#fb047642
```

## Acknowledgements / Further reading

This proposal builds on the contributions from the community.

Specifically [this proposal is also based on an investigation by HMRC](https://gist.github.com/rpowis/71f3782166e7d835b12ffe7740f6b23e#3-prepend-a-directory-at-the-package-root)

You can see the [original discussion on GitHub](https://github.com/alphagov/govuk-frontend/issues/870).
