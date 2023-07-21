# Remove `init()` method from JavaScript components
<!-- The title should be a short present tense imperative phrase, less than 50
     characters, like a git commit message. -->

**Date:** 2023-07-20

**Status:** Accepted

## Decision

<!-- An overview of the decision made. -->

We'll be removing the `init()` method from our JavaScript components,
in favour of them initialising automatically during instantiation.

This simplifies individual component's initialisation from:

```js
new Accordion($element).init()
```

To:

```js
new Accordion($element)
```

## Rationale

The `init()` method adds an unnecessary step for our components' initialisation, without tangible benefits for what our components do. Without it we can simplify both how our components are initialised, as well as their internal implementation as we're looking to [throw errors at instantiation](./009-throw-errors-component-constructors.md).

The `init()` method enables the following, that none of our components benefit from, so we can remove that extra step:

- delaying the initialisation of the component after its instantiation
- allowing the initialisation code to be `async` (which a `constructor` cannot be)

With our intent to [throw errors at construction](./009-throw-errors-component-constructors.md), having a separate `init()` method requires careful separation between validation of the component in the `constructor()` and initialisation code in a subsequent call. By having only the constructor, the initialisation code can simply `throw` from within `constructor()` to ensure users always get functional instances of our components. 

<!-- The reason for the decision and any comments or disagreements cited. -->

## Risks and constraints

### Reduce or delay the adoption of v5

This change requires code changes from our users, that they may not have the capacity to implement.

This is mitigated by the changes being quite small (removing a `.init()` call following the current instantiation of the component) and [only affects a portion of our users](#users-need-to-update-their-code), who use our component classes directly rather than through `initAll()`.

<!-- Any accepted risks associated with the decision. -->

## Alternatives considered

### Deprecating the init method in v5 for future removal in v6

Because there wasn't really an alternative for users to switch to if we'd deprecated in v4 (if people stopped calling `init()`, there'd be no initialisation), we couldn't make the deprecation in v4.

Implementing an empty `init()` to document its deprecation in v5 would only delay the update users need to make in their code. This will come at the detriment of the next release: on top of focusing on the topic of that new release, people will also have to update their JavaScript (which is the focus of v5).

The scope of the change users have to make, which will be documented, as well as our continued support for v4 during a year after we ship v5 seems enough of a mitigation to directly remove the method.

### Keeping the `init` method to make use of `async`

The `init()` method could be made asynchronous, which would isolate the initialisation of each component and allow it to wait for asynchronous calls without blocking the main thread.

However, since:

- Asynchronous code requires a deeper understanding of concurrency
- Deferred errors from Promises must be awaited before they throw
- None of our components make asynchronous calls

We can currently consider the `init()` method unnecessary. That said, we may add asynchronous methods to our components in future.

<!-- Other options discussed but decided against. -->

## Implications

<!-- Impacts of this decision. -->

### Users need to update their code

Component initialisation will be reduced to `new Example()` instead of `new Example().init()`. Only users that call component `init()` themselves will need to update their code to reflect this.

The `initAll()` method will not be affected.

## Contributors

- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

<!-- Add links to any related GitHub issues and / or PRs. -->

## Outcomes

<!-- Note any outcomes or consequences of the decision . -->
- Implement the removal of the `init()` method for each of our component
- Document the code update `govuk-frontend` users will need to implement following our removal 
