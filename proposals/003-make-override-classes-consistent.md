# Make override classes consistent

**Closing date:** N/A

**Status:** Accepted (originally raised as a [gist](https://gist.github.com/nickcolley/f135e89ed4b679355b0ab47135b38ee8))

## Summary

We're getting close to 1.0.0 and are tidying up and loose ends for the public API.

One of the final things we're auditing is the consistency between class names,
we believe if they're more consistent they'll be easier to remember and more intuative.

## Problem

Right now our override classes look like:

```
.govuk-!-display-inline {}
.govuk-!-display-inline-block {}
.govuk-!-display-block {}
.govuk-!-m-rN {}
.govuk-!-p-rN {}
.govuk-!-f-N {}
.govuk-!-w-regular {}
.govuk-!-w-bold {}
.govuk-!-width-three-quarters {}
.govuk-!-width-two-thirds {}
.govuk-!-width-one-half {}
.govuk-!-width-one-third {}
.govuk-!-width-one-quarter {}
```

(Note for sake of demonstration I've not included all the [spacing classes](https://design-system.service.gov.uk/styles/spacing/#using-the-override-classes))

Some examples of the short hand classes are:

- `.govuk-!-w-bold` which sets the font weight to 'bold')
- `.govuk-!-f-30` which sets the font size to 24 (responsively)
- `.govuk-!-mb-r3` which sets the margin bottom to the spacing scale of 3 (responsively)

## Proposal

We're considering making all overrides consistent, this would look like either:

### Option 1: Consistently longhand

```
.govuk-!-display-inline {}
.govuk-!-display-inline-block {}
.govuk-!-display-block {}
.govuk-!-margin-5 {}
.govuk-!-margin-bottom-5 {}
.govuk-!-padding-0 {}
.govuk-!-padding-bottom-0 {}
.govuk-!-font-size-30 {}
.govuk-!-font-weight-regular {}
.govuk-!-font-weight-bold {}
.govuk-!-width-three-quarters {}
.govuk-!-width-two-thirds {}
.govuk-!-width-one-half {}
.govuk-!-width-one-third {}
.govuk-!-width-one-quarter {}
```

#### Pros
- easier to understand what the classes are doing when seeing them for the first time

#### Cons
- since override classes tend to be used together, verbose class names could make markup verbose

### Option 2: Consistently shorthand

```
.govuk-!-d-i {}
.govuk-!-d-ib {}
.govuk-!-d-b {}
.govuk-!-m-5 {}
.govuk-!-mb-5 {}
.govuk-!-p-0 {}
.govuk-!-pb-0 {}
.govuk-!-fs-30 {}
.govuk-!-fw-regular {}
.govuk-!-fw-bold {}
.govuk-!-w-3/4 {}
.govuk-!-w-2/3 {}
.govuk-!-w-1/2 {}
.govuk-!-w-1/3 {}
.govuk-!-w-1/4 {}
```

#### Pros
- once the naming convention is understood, it is faster to type
- less verbose when composing classes together.

#### Cons
- higher overhead to learn and remember this naming convention

### Composing classes
To explore what it would look like to use the long hand I took some real examples of using overrides from the GOV.UK Design System and put them side by side.


#### Service unavailable pages example

No change:

```html
<h1 class="govuk-heading-xl">Service unavailable</h1>
<p class="govuk-body-l">The submit an employment intermediary report service has closed.</p>
<p class="govuk-body">Contact us if you need to speak to someone about the service and your reports.</p>
<p class="govuk-body govuk-!-mb-r0">Telephone:</p>
<p class="govuk-body govuk-!-w-bold">0808 157 3900</p>
<p class="govuk-body govuk-!-mb-r0">Opening times:</p>
<p class="govuk-body govuk-!-w-bold">Monday to Friday: 8:30am to 4:30pm</p>
```

Expanded:

```html
<h1 class="govuk-heading-xl">Service unavailable</h1>
<p class="govuk-body-l">The submit an employment intermediary report service has closed.</p>
<p class="govuk-body">Contact us if you need to speak to someone about the service and your reports.</p>
<p class="govuk-body govuk-!-margin-bottom-0">Telephone:</p>
<p class="govuk-body govuk-!-font-weight-bold">0808 157 3900</p>
<p class="govuk-body govuk-!-margin-bottom-0">Opening times:</p>
<p class="govuk-body govuk-!-font-weight-bold">Monday to Friday: 8:30am to 4:30pm</p>
```

#### Phase banner example

https://github.com/alphagov/govuk-design-system/blob/cf049adba070d349513313af43ad962fde16914c/views/partials/_banner.njk


No change:

```javascript
{{ govukPhaseBanner({
  tag: {
    text: "preview",
    classes: "app-tag--review"
  },
  classes: "govuk-!-pl-r3 govuk-!-pr-r3",
  html: phaseBannerText
}) }}
```

Expanded:

```javascript
{{ govukPhaseBanner({
  tag: {
    text: "preview",
    classes: "app-tag--review"
  },
  classes: "govuk-!-padding-left-3 govuk-!-padding-right-3",
  html: phaseBannerText
}) }}
```

#### Start page example

https://github.com/alphagov/govuk-design-system/blob/cf049adba070d349513313af43ad962fde16914c/src/design-system/patterns/start-pages/default/index.njk

No change:

```html
<p class="govuk-body">Registering takes around 5 minutes.</p>

{{ govukButton({
  text: "Start now",
  href: "#",
  classes: "govuk-button--start govuk-!-mt-r2 govuk-!-mb-r8"
}) }}

<h2 class="govuk-heading-m">Before you start</h2>
```

Expanded:

```html
<p class="govuk-body">Registering takes around 5 minutes.</p>

{{ govukButton({
  text: "Start now",
  href: "#",
  classes: "govuk-button--start govuk-!-margin-top-2 govuk-!-margin-bottom-8"
}) }}

<h2 class="govuk-heading-m">Before you start</h2>
```

### Related
- https://tachyons.io/ (https://github.com/tachyons-css/tachyons-verbose)
- http://basscss.com

What do you think? Should we keep these overrides as they are? Or should we make them consistent one way or the other?
