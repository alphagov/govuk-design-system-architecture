# Consistently initialise JavaScript using `govuk-` namespaced data-module attributes

**Closing date:** 2019-06-11

**Status:** accepted

## Summary

We want to change the way that JavaScript in GOV.UK Frontend is initialised against elements in the DOM.

We're doing this to help users avoid conflicts in their application.

This would be a breaking change in the 3.0.0 release.

## Problem

Currently to initialise the JavaScript for a component we look for a
`data-module` attribute on the element.

For example, the accordion HTML looks like this:

```html
<div class="govuk-accordion" data-module="accordion">
  [..]
</div>
```

And we initialise the JavaScript like this:

```javascript
// Find all global accordion components to enhance.
var $accordions = scope.querySelectorAll('[data-module="accordion"]')
$accordions.forEach($accordion =>  {
  new Accordion($accordion).init()
})
```

This works, but there's a risk of conflict if `[data-module="accordion"]` is used by anything else in the application.

This [happened on GOV.UK](https://github.com/alphagov/collections/pull/1042), resulting in two 'open all' links appearing on a single accordion.

---

Not all components are initialised in this way. The JavaScript for the button component, for example, sets up event listeners against the `scope` (usually `document`).

This is inconsistent, and means that some components have a higher chance of conflicting as they e.g. affect anything with `role="button"`

## Proposal

We intend to move to using a `govuk-` namespace for the `data-module` attribute.

```html
<div class="govuk-accordion" data-module="govuk-accordion">
  [..]
</div>
```

```javascript
// Find all global accordion components to enhance.
var $accordions = scope.querySelectorAll('[data-module="govuk-accordion"]')
$accordions.forEach($accordion =>  {
  new Accordion($accordion).init()
})
```

We also intend to initialise all components that use JavaScript in the same way.

For example, the JavaScript for buttons would be updated to initialise against `[data-module="govuk-button"]`.

```html
<a href="/" role="button" data-module="govuk-button" class="govuk-button">Continue</a>
```

```js
var $buttons = scope.querySelectorAll('[data-module="govuk-button"]')
$buttons.forEach($button =>  {
  new Button($button).init()
})
```
