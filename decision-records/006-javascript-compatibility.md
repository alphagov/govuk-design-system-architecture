# JavaScript browser compatibility

Date: 27/06/23

Status: Accepted

## Decision

We will publish JavaScript that can be parsed without errors in browsers supporting `<script type="module">`.

We will update our guidance to tell service teams to include the JavaScript from GOV.UK Frontend using `<script type="module">`.

We will set up:

- transpiling modern JavaScript to a syntax parsable by [browsers that support ES modules](https://browsersl.ist/#q=supports+es6-module)
- linting to prevent us from using newer features and syntaxes unless we have actively opted in to them

Going forward, when we want to use a newer feature we will disable the linting rule only after checking the performance impact of the transformation.

Our current components do not require any polyfills. We set up the transpilation to only make [syntax transformations](https://babeljs.io/docs/babel-preset-env) and [bundle (to avoid having a dependency) Babel helpers](https://github.com/rollup/plugins/tree/master/packages/babel#babelhelpers), not automatically [inject third party polyfills](https://github.com/babel/babel-polyfills). We will revisit this when we want to introduce a new feature that would require polyfills.

In order to keep only style components as interactive when the JavaScript is going to be included, we will [add a new `govuk-frontend-supported` class to the body](https://github.com/alphagov/govuk-frontend/pull/3801) as part of the inline script if the browser supports `HTMLScriptElement.prototype.noModule`, and add a guard clause that checks for the class to the initialisation of each component.

## Rationale

In line with our agreed browser support, we want to ensure that the JavaScript for GOV.UK Frontend is only loaded and executed if it is likely to be parsed and executed successfully.

As we're supporting a more modern set of browsers, and tooling has evolved since [our last decision on JavaScript compatibility](./001-javascript-for-less-capable-browsers.md), we're updating our strategy to fulfil that goal.

We believe this provides the right balance between engineering effort and performance, whilst maximising the likelihood that regardless of what browser a user is using, they will be able to complete their task.

## Risks and constraints

### Requires a good non-JavaScript experience

As JavaScript will only execute in browsers that support `<script type="module">`, our approach requires our components to follow the principles of progressive enhancement to ensure users can still complete their tasks in browsers that don't support it.

Similarly for services that bundle their JavaScript with GOV.UK Frontend, their own features will need to be implemented following the principles of progressive enhancement as well.

### Requires teams to follow our guidance

Loading the JavaScript only in browsers where it will execute without errors requires services to use `type="module"` on their script tags, which we have no control over.

### May require services to refactor their JavaScript

Some services may need to run code in all browsers (rather than only those supporting `type="module"`), for example analytics. This code would need to be split from the code that bundles GOV.UK Frontend, requiring services to refactor their code accordingly.

### Decision overhead

We're planning to opt-in to the use of specific modern features only after assessing their impact on the transformed code. This might lead to an overhead for researching and documenting that impact, as well as making the decision of whether to opt-in or not.

### Tooling isn't comprehensive

We're not sure of the exact coverage our tools will provide for either checking the use of features or transforming them down to ES 2015.

For example, [eslint-plugin-es-x](https://eslint-community.github.io/eslint-plugin-es-x/) does not cover Web APIs and there's an engineering cost to [using the transpilation step for detecting whether polyfills would need including](https://github.com/babel/babel-polyfills/pull/13/files) reliably (especially if we guard the execution of some of our features to the presence of specific APIs in the browsers, for which we wouldn't want a polyfill).

This means we still need some level of manual checks, which are susceptible to human errors.

### Tooling evolution

The tools we plan to use may evolve to only support a more modern baseline than the support of ES modules (similarly to how Rollup stopped supporting Internet Explorer 8 at some point). This could force us to pin our tools to a specific version if we want to keep benefiting from them, devise a new approach for ensuring our compatibility, or revisit our baseline (this would require a major release though).

### Published code monitoring

By making further transformations of our source code into published code, we potentially provide more room for malicious code to be introduced by our tooling (supply chain attacks).

Monitoring the changes from one release to another may also be trickier as tooling updates may introduce extra changes to the code (for example refactoring or reordering...) that would make comparison harder.

## Alternatives considered

### Adding the styling hook using type="module"

The inline script adding the `govuk-frontend-supported` class cannot be run in a `<script type="module">`. This would defer the execution of the script adding the class until after the page loads. This defeats the purpose of that script, as it needs to run immediately so it prevents flashes of non-interactive styles.

### Publishing our source code

We could write source code which browsers supporting ES modules can parse, and publish it without making any transformations.

While we could lint away some compatibility concerns, this would require more manual testing to ensure we match what the browsers we support can parse and execute.

Due to the more manual nature, it would also be more prone to bugs and regressions.

And finally, due to the extra knowledge necessary to know what's compatible or not, it would require extra effort from developers and contributors to offer changes.

### Guarding based on newer APIs and letting JavaScript error

Instead of using the availability of `HTMLScriptElement.prototype.noModule` to set the `govuk-frontend-supported` class (which governs both styling and execution of our components), we could:

- pick a more modern feature to check (for example `Promise.prototype.finally`)
- transpile down to the more modern set of browsers that support it (rather than all the way down to browsers that support ES modules)
- and let JavaScript error in browsers that support `type="module"` but not the newer features that appear in the code we publish

This would further reduce the amount of features that get transpiled (most notable would be spread `...` and `async/await` if we chose `Promise.prototype.finally`). However this comes with the following drawbacks:

- The extra errors in browsers that support `type="module"` but not our new baseline makes it harder to know if the error is expected (because the browser is not supported) or not. This complicates both the debugging for our users and the triaging of bugs for us.
- Because of syntax errors, service code bundled with GOV.UK Frontend would not get a chance to be executed at all. This would require extra effort on the services part to decide what can be bundled with GOV.UK Frontend and what needs to be in a separate script to ensure it runs.

### Recommending dynamic import of GOV.UK Frontend

Using a dynamic `import()` call would allow to handle syntax errors when parsing GOV.UK Frontend (via a `.catch` on the promise, or a `try/catch` block, depending on the browsers you'd want to support).

This complicates the integration for services as it requires more JavaScript knowledge to understand which piece of code will run or not if GOV.UK Frontend fails to parse. It also complicates the import of multiple individual components, requiring to make sure the `import()` calls run in parallel to not hinder performance (on top of the individual components not being discoverable by [the browsers' preload-scanner](https://web.dev/preload-scanner/)), but handle the potential failure of these calls appropriately.

### Import maps

Import maps could provide another way to only load GOV.UK Frontend in browsers with a certain level of support for modern JavaScript features. It's [not widely supported enough yet](https://caniuse.com/import-maps), but could be a good consideration if we revisit which baseline we want to have in the future.

## Implications

### Automating parts of the development process

When writing or changing JavaScript we currently have to remember not to use features that aren't available in the oldest browsers we support, or to manually add a polyfill. We also rely heavily on manual testing across multiple browsers.

Improving our build process by introducing transpilation and additional linting rules:

- improves the developer experience, as we provide additional safety guards against using unsupported features
- enables us to use a modern syntax, including future features as long as our tooling can transpile it

At the same time, we need to recognise that we are deferring some of the responsibility of ensuring browser compatibility to our linting and transpilation tools.

If we find that the transpilation is not producing code that works in the targeted browsers, we may be relying on the maintainers of those dependencies to help us address issues.

We will also need to carefully monitor the impact of transpilation as the code we publish will potentially be further removed from our source code.

### Improving the experience on older devices

Restricting JavaScript to running only on browsers that support `type="module"` means that it will not run on the oldest browsers, which are likely to be on old, underpowered devices. This means that users on older devices will get a simplified, but likely more performant experience.

### Pins the JavaScript we publish to 2017

This approach pins us to publishing code based on browsers that were released in 2017.

With the exception of Internet Explorer 11 (for which we've decided to not provide JavaScript enhancements for), [this target](https://browsersl.ist/#q=supports+es6-module) covers further than [the 95%](https://www.gov.uk/service-manual/technology/designing-for-different-browsers-and-devices#understanding-the-table) of [visits to GOV.UK (internal link)](https://docs.google.com/presentation/d/17gcoKyQy6YnfcBG6LGIkNKi6VVOz9nc_EffvA39o98Y/edit#slide=id.g1bee1a35b37_0_83).

The target creates a long tail of browsers that users no longer use and for which we'll publish code transformations. This long tail will only expand as browser vendors release newer versions.

We need to be mindful that:

- some of the transformations may increase the file size of the JS to provide compatibility to only a tiny percentage of users
- the number of transformations required will grow over time as new JS features are introduced
- we may be prevented from using new JS features if we think the performance implications of transpiling are too great

Pinning to browsers that support ES Modules will only make sense for so long. We should revisit this if either:

- the extra weight caused by the transformations and polyfills becomes untenable
- we are unable to implement something that would provide value for end users (either because there is no way to transpile it, or because it would add too much weight)
- a better way to do this becomes available

This will require service teams to make further changes.

We are working on the assumption that in the future there will be other mechanisms we can use to provide this 'clean cut'.

## Contributors

- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

## Outcomes

- Set up Babel
- Update linting to ES2015
- Performance monitoring
