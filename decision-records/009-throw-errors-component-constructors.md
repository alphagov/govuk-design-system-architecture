# Throw errors from component constructors

**Date:** 2023-07-20

**Status:** Accepted

## Decision

In v5.0, we will update the constructor of every component to throw errors if the arguments passed to the constructor are invalid, rather than silently returning.

## Rationale

Currently, if we encounter something unexpected during instantiation of an object we return early. For example:

```js
if (!($module instanceof HTMLElement) || !document.body.classList.contains('govuk-frontend-supported')) {
  return this
}
```

This can make unexpected behaviour difficult to debug for users – because we silently return, the user has no idea why the accordion isn't doing anything.

The user also has a reference to an instantiated component in an invalid state.

As we introduce new methods to the public API, we would need to handle the possibility that the component is in an invalid state as part of every individual method:

```js
const accordion = new Accordion(aNonExistentElement)
accordion.openAllSections() // what do we do here?!
```

Throwing an error from the constructor means that:

- we are following the 'fail fast' principle
- any unexpected behaviour is easier to debug
- the user can never have a reference to an instantiated component that threw exceptions from the constructor (because the assignment will fail), which means we can assume the component is in a valid state when writing other public methods

## Risks and constraints

### Changes prevent some users from upgrading

Users who are relying on being able to access now private class or instance properties from from their own code will need to either:

- wait until we introduce an equivalent way to achieve the same thing using the public API before they can upgrade beyond v5
- continue to use the private properties 'at risk', and manually test or verify changes between releases

### Changes cause unhandled JavaScript errors in services

If thrown exceptions are left unhandled this will stop script execution, including JavaScript for other components on the page. Some of our components can [fail closed if JavaScript is enabled in the browser but the script fails to run](https://github.com/alphagov/govuk-frontend/issues/999).

We will treat this as a breaking change and ensure that it is well documented in the release notes.

This doesn't mean that introducing additional scenarios where the constructor may throw will always be breaking changes. The breaking change is that it's now possible that the constructor may throw, when it never would before.

## Alternatives considered

None.

## Implications

If we see users struggling with error handling when instantiating instances we may consider introducing a 'fail-safe' way of instantiating, for example using a static method on the class that simply logs errors to the console.

## Contributors

- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

<!-- Add links to any related GitHub issues and / or PRs. -->

## Outcomes

- Update the constructors of each component to throw errors if the arguments passed to the constructor are invalid
