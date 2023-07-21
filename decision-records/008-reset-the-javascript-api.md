# Reset the JavaScript API for our components

**Date:** 2023-07-20

**Status:** Accepted

## Decision

In v4.x, for each component class, we will mark all class and instance properties other than the constructor and the `init` method as deprecated.

In v5.0 we will then add JSDoc annotations to mark the deprecated properties as private. We will also annotate any 'internal' classes or functions (like the `I18n` class and the `mergeConfigs` function) as internal.

In future releases we will then gradually introduce a new public JavaScript API for components, which is:

- based on the needs of users
- clearly defined and documented
- comprehensively tested

## Rationale

The JavaScript API for our components has not been designed to be called externally and does not clearly differentiate between public and private methods.

Many of the methods exposed on the classes or objects can only be called internally as they need to be passed internal state, or are designed to be used internally by the component as event handlers.

We also have little or no test coverage for calling these methods externally.

This means that:

- it's unclear for users whether they are allowed to call or interact with each property
- it's difficult for us to judge what constitutes a breaking change

'Resetting' the public API by marking everything else as private gives us the freedom to refactor the internals of the component before then introducing a properly designed, tested and documented public API.

## Risks and constraints

Users who are relying on being able to access now private class or instance properties from from their own code will need to either:

- wait until we introduce an equivalent way to achieve the same thing using the public API before they can upgrade beyond v5
- continue to use the private properties 'at risk', and manually test or verify changes between releases

## Alternatives considered

### Do nothing

We could leave the API as ambiguous, however we know that following other recent major changes to the way that we write JavaScript, we will want to be able to refactor the components to take advantage of newer JavaScript features and to reduce the amount of duplication between components.

Doing this without a clear definition of the public API will be difficult, as it will be hard to reason whether a particular change is breaking or not.

### Maintain the existing API

We could aim to introduce a new full-featured public API before we remove any of the existing properties. However, it would be difficult to do this without significant refactoring of the existing code, which would almost certainly require the removal or renaming of existing methods and variables. We would also need to introduce comprehensive testing of the existing methods to ensure their behaviour was maintained.

Aside from the constructor and the `init`` method, none of the existing class or object properties have been designed to be called externally or documented. We believe that the vast majority of our users only interact with the constructor and init methods.

Therefore we believe the additional engineering effort required to preserve the existing 'accidental' API is not justified.

## Contributors

- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

- [#3478: Clarify our JavaScript components' public API](https://github.com/alphagov/govuk-frontend/issues/3478)

## Outcomes

1. In v4.x, deprecate all properties except the constructor and the init method.
2. In v5.0, mark the deprecated properties as private.
3. In v5.0, mark internal classes and functions as internal.
4. In future releases, gradually introduce a properly designed API.
