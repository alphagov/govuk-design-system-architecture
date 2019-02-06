# Enable importing CSS Modules via filename convention used in common frameworks

**Closing date:** 2019-02-06
**Status:** Rejected

## Summary

This proposal is a continuation of the [initial conversation raised on the GOV.UK Frontend repository](https://github.com/alphagov/govuk-frontend/issues/1052).

[CSS Modules](https://github.com/css-modules/css-modules) is a standard way to allow users to import CSS into JavaScript files (commonly known as CSS-in-JS).

Files imported as CSS Modules automatically are encapsulated by the build tool.

In order to differentiate between files that should be imported as regular CSS and CSS modules some popular frameworks use a filename convention ‘filename.module.css’.

This means that end users do not need to provide custom setup to a build tool such as webpack to import the file as a CSS Module.

Examples of these projects include:

- [create-react-app](https://facebook.github.io/create-react-app/docs/adding-a-css-modules-stylesheet)
- [Neutrino](https://neutrinojs.org/packages/web/)
- [Gatsby](https://github.com/gatsbyjs/gatsby)
 
This may become a standard convention recommended by the CSS Modules specification: [css-modules/css-modules#229](https://github.com/css-modules/css-modules/issues/229)

## Problem

Users can currently import GOV.UK Frontend files into components using a standard import, for example using the [webpack css loader](https://github.com/webpack-contrib/css-loader).

Downstream projects such as [govuk-react](https://github.com/govuk-react/govuk-react) want to import only the files necessary for each component, then publish these components as separate npm packages.

If a project like this imported CSS with a default loader conflicts would appear if the components use different versions of GOV.UK Frontend.  
  
In popular frameworks this is the default unless you use the `filename.module.css` convention, which automatically encapsulates classes.

Multiple packages is something we [decided against when publishing GOV.UK Frontend](https://github.com/alphagov/govuk-design-system-architecture/blob/master/proposals/002-publish-one-npm-package-instead-of-multiple-npm-packages.md) as we are not sure people having inconsistent parts of the system would be a good idea, and multiple packages is not a requirement to have modular builds (including code splitting).

However we can’t anticipate how people might want to use GOV.UK Frontend in the future.

One issue that may arise from a project such as govuk-react trying to use CSS Modules will be that [some of our component CSS selectors reference other components](https://github.com/penx/govuk-frontend/blob/37e44dd2dacecb956299a129c5766aebaed40c06/src/components/character-count/_character-count.scss#L9) CSS selectors, which may not work.  

After speaking to Alasdair McLeay (lead of govuk-react) we agreed that to make components like the Character Count work correctly with CSS Modules we’d have to change the markup, resulting in a breaking change for our other users.

With this in mind we think CSS Modules will for the most part still be useful for projects in the future and if this form of using CSS becomes more popular in the future we can consider making breaking changes to improve interoperability with CSS Modules but this should not block implementing an imperfect support.

## Proposal


We would broadly implement support for CSS Modules by doing the following:

### 1a. Option 1: Manually create new `.module.scss` entry points for all SCSS files that output CSS

- Core
- Objects
- Components
- Utilities
- Overrides

For example:

```scss
// header.module.scss

@import “./header.scss”  
```

Benefit of this options is it avoids any opaque build step transforms and allows for reasonable adjustments to the module files in the future if necessary.

### 1b. Option 2: Automatically create new `.module.scss` entry points for all SCSS files that output CSS with duplicate content

- Core
- Objects
- Components
- Utilities
- Overrides

Benefit of this option is the source directory is simpler.

### 2. Write guidance in the govuk-frontend installation documentation indicating that we support this CSS Modules filename convention

### Guidance

Since we’re worried that many tools that use CSS Modules also do not support [building projects to the standards we expect](https://www.gov.uk/service-manual/technology/using-progressive-enhancement), we will take a neutral stance in any guidance detailing this functionality to avoid accidentally promoting specific frameworks.

We would also make a note that this is an alpha / experimental feature.

### Pros
- Projects such as govuk-react can have more confidence when their projects are used in common tools that support the CSS Modules file name convention.
- Adds additional future proofing to GOV.UK Frontend

### Cons

#### GOV.UK Frontend is not designed to be imported via CSS Modules

Some parts of GOV.UK Frontend are not written in a way that work out the box with css modules, and the fixes necessary would be breaking changes for all users.

#### Duplicate files in published package could be confusing

If we ensure the guidance is clear we don’t have to make users make a decision and only make CSS Modules a feature for people looking for it.

#### Risk that recommend tools that don't meet our standards

We are going to have a neutral stance on css modules in relation to frameworks/tools/bundlers, and leave it to teams to pick the right tools for what they’re building.

#### Option 2 only: A bigger npm package could mean slower npm installs for all users

We think the risk of this is low as the files uploaded to npm are first compressed.

Since duplicated plain text files are easy to compress we hope this would mean the increased size would not impact install time by a large amount.

We can confirm this by investigating using `npm pack` will create a compressed zip which is the same as published to npm.

#### Option 2 only: makes the build pipeline more complicated: takes longer, src gets further away from package

While it does add complexity, it would mostly involve copying and renaming files so we think it’s not a big risk.
