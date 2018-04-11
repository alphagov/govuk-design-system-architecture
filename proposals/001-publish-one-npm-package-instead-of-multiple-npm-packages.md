# Publish one npm package instead of multiple npm packages

**Closing date:** 2018-03-27

**Status:** Accepted

## Contents
- [Summary](#summary)
  - [Current architecture](#current-architecture)
- [Problems associated with publishing individual packages](#problems-associated-with-publishing-individual-packages)
  - [Versioning](#versioning)
  - [Conflicts](#conflicts)
    - [Pancake](#pancake)
- [Proposal: Publishing one package](#proposal-publishing-one-package)
  - [Currently](#currently)
    - [Steps to build everything](#steps-to-build-everything)
    - [Steps to build only one component](#steps-to-build-only-one-component)
  - [After proposed change](#after-proposed-change)
    - [Steps to build everything](#steps-to-build-everything)
    - [Steps to build only one component](#steps-to-build-only-one-component)
  - [Risks with moving to a single package](#risks-with-moving-to-a-single-package)
    - [Breaking changes that impact many packages could make it difficult to upgrade only parts at a time.](#breaking-changes-that-impact-many-packages-could-make-it-difficult-to-upgrade-only-parts-at-a-time)
- [Options](#options)
- [Reference](#reference)
  - [Examples of modular projects that use single packages](#examples-of-modular-projects-that-use-single-packages)
  - [Examples of modular projects that use multiple packages](#examples-of-modular-projects-that-use-multiple-packages)
- [With thanks to](#with-thanks-to)


## Summary

Publish one package with all of GOV.UK Frontend while still allowing for building individual components only.

GOV.UK Frontend is currently made up of multiple npm packages such as “date-input” and “footer” (see [packages/](https://github.com/alphagov/govuk-frontend/tree/master/packages)).

This proposal is interested in doing the minimum in order to ship a public release of GOV.UK Frontend.
With the hope that this decision can be changed in the future given more time or better evidence of the need for this.

### Current architecture

GOV.UK Frontend can be consumed in two main ways:
- All at once
- By individual components

Allowing for individual components is important to:
- Test that they are isolated and do not 'leak' and break other components / elements on a page.
- Makes it easier to deprecate and remove in the future.
- Easier to port to [insert your favourite templating approach here]
- Make it easy to upgrade parts at a time, for example only the new(er) radio components.
- Have smaller builds that only include what is used.

We have two main aspects of the architecture that allows for this.

1. Modular project structure
2. Individually published packages to npm

## Problems associated with publishing individual packages

### Versioning
We do not currently have a process to version our individual components before publishing them. If we decide we want to persist with the multiple packages approach, we’ll need to work this out.

Our current process is to give all packages the same version in lockstep.

This means all our packages have the same version number, regardless if they have actually changed in a release.

### Conflicts

This is explained really well in [npm and the front end] (thanks Dominik Wilkowski! 💌)

Currently `button` component and `input` component both depend on the `label` component.

```bash
label
├── button
└── input
```

In the current case, these are all versioned in lock-step so when you install the button and input you get a dependency tree that looks like the following:

```bash
npm install --save button input
```
```bash
label@1.0.0
├── button@1.0.0
└── input@1.0.0
```

However if we made a breaking change to the `label` component, the `button`  and `input` component may also require a breaking version update.

If a user then installs only the `input` component update, this is where we run into issues:

```bash
npm install --save button@1.0.0 input@2.0.0
```
```bash
label@1.0.0
└── button@1.0.0

label@2.0.0
└── input@2.0.0
```

Now we have a duplication of the label component, and since CSS is a global language depending on how they are imported into a project it'd result in a broken behaviour.

We use an 'export' pattern that ensures an imported scss partial is only included once (https://github.com/alphagov/govuk-frontend/blob/e583c8b3e4038c09c53d4ffbbc129ea8495efbe9/src/globals/tools/_exports.scss)

```css
.govuk-c-label { ... } /* @2.0.0 */
.govuk-c-input { ... } /* @2.0.0 */
.govuk-c-button { ... } /* @1.0.0 */
```

As explained in [npm and the front end] npm was designed in a world where this is not a huge concern.
Since backend code can have different versions of the same dependency and not have any global conflicts.
This duplicate backend code is also not sent over the wire and run by users.

### Pancake
[Pancake] is a really interesting solution to the above problem built on top of npm, it works around the many issues around `peerDependencies` (see the [npm blog post])

However, we found in research that many people have big existing build pipelines so adding more tools could be a problem.

We’d need to do more research to see if pancake would make these problems worse.

At the moment we’re not clear that the time invested would be worth the problems it solves, however this proposal assumes we could move towards a system like pancake from a single package if a clearer need was presented to us.

See below for ‘problems with a single package’.

## Proposal: Publishing one package

### Currently

#### Steps to build everything
```bash
$ npm install --save @govuk-frontend/all
```

```scss
// application.scss
@import "@govuk-frontend/all/all";
```

```bash
$ node-sass --include-path=node_modules application.scss > application.css
```

#### Steps to build only one component
```bash
$ npm install --save @govuk-frontend/button
```

```scss
// button.scss
@import "@govuk-frontend/button/button";
```

```bash
$ node-sass --include-path=node_modules button.scss > button.css
```

### After proposed change

https://github.com/alphagov/govuk-frontend/compare/spike-rfc-packages

#### Steps to build everything
```bash
$ npm install --save @govuk-frontend
```

```scss
// application.scss
@import "@govuk-frontend/all/all.scss";
```

```bash
$ node-sass --include-path=node_modules application.scss > application.css
```

#### Steps to build only one component
```bash
$ npm install --save @govuk-frontend
```

```scss
// button.scss
@import "@govuk-frontend/button/button.scss";
```

```bash
$ node-sass --include-path=node_modules button.scss > button.css
```

This also means when moving from individual components to multiple components,
you only need to add additional `@import` statements, and would not need to run `npm install` again.

### Risks with moving to a single package

The time it takes to install a big package is longer than individual packages.
Individual packages force us to keep code modular, we would need to ensure our testing makes up for this.
Mental model of custom builds may not be as well understood, we need to do more to make it clear you don’t need the whole project.

#### Breaking changes that impact many packages could make it difficult to upgrade only parts at a time.

> “If you’re still creating one version for the entire thing, then when you fix an accessibility bug in one component everyone will have to update the entire thing.” - Dominik Wilkowski GOV.AU

> “We come from a monolithic CSS (Bootstrap), and the reason for us moving to individual components was that it reduces ‘upgrade anxiety’ and helps with adoption. “ - Dominik Wilkowski GOV.AU

> "Each component has an API. This API is a combination of CSS classes, HTML structure, Sass mixins, and JavaScript methods. Whenever any of these APIs needs to be deprecated, we release a new Node module with a new version by following semantic versioning.

> These sorts of breaking changes require our clients to change THEIR code before they can upgrade to the new version.

>By versioning each component separately....YOU are in control of when you upgrade each component. This gives you flexibility in your timeline. If you are under some time constraint to launch a new feature, then you might wait to upgrade the component.

> You don't have to feel pressured to upgrade everything right away." - Lynn Jespen, Tech Lead for Material Components Web (via Jani Kraner)

This has come up multiple times, potentially the risk of this could be improved by releasing often and not ‘save all the big changes’ for a breaking change.

For example, if we know we’ll be making a breaking change to Component 1 and Component 2. Release Component 1 as a new breaking change, then do another release for Component 2.

Note: In this case if you only want updates for Component 2, you’ll be forced to also bring in updates for Component 1.

With a release process that encourages releasing after merging to master this could be possible.

Without this it would require careful orchestration of releases which could create blockers in our workflow.

It would also mean we would need to do more to communicate breaking changes that are big fundamental changes, this could be done with release names (see ubuntu for example of ‘code names’ https://wiki.ubuntu.com/Releases)

This seems to be the biggest risk to publishing one package.

Please contribute with any risks we have not considered here.

## Options

- Continue with current approach: Flatten and remove conflicts after install ([pancake])
- Approve the proposed change: Publish one package

### Reference

- Transcript of the conversation with the GOV.AU team: https://gist.github.com/nickcolley/eb61fedb20d273a5266cdf4f71cfbb30
- npm blog post introducing the problems relating to the issues described in this RFC: http://blog.npmjs.org/post/101775448305/npm-and-front-end-packaging
- Blog post from Dominik explaining their solution to this problem: https://medium.com/dailyjs/npm-and-the-front-end-950c79fc22ce
- Pancake: https://github.com/govau/pancake/

#### Examples of modular projects that use single packages
- Bootstrap - High barrier to entry https://getbootstrap.com/docs/4.0/getting-started/build-tools/#tooling-setup
- Foundation - Much better documentation https://foundation.zurb.com/sites/docs/sass.html#adjusting-css-output
- Carbon Design System - http://carbondesignsystem.com/getting-started/developers

#### Examples of modular projects that use multiple packages
- https://github.com/primer/primer
- https://github.com/ElemeFE/mint-ui - Instructions make no mention of installing individual packages
- https://github.com/fyndiq/fyndiq-ui
- https://github.com/material-components/material-components-web
- https://github.com/govau/uikit

## With thanks to
- GOV.AU team for spending time with us to talk through their decisions for their project.
- Lynn Jespen, Tech Lead for Material Components Web
- GOV.UK Design System team

[npm blog post]: http://blog.npmjs.org/post/101775448305/npm-and-front-end-packaging
[npm and the front end]: https://medium.com/dailyjs/npm-and-the-front-end-950c79fc22ce
[pancake]: https://github.com/govau/pancake/
