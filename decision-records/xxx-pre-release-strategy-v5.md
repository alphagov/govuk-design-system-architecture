# Pre-release strategy for v5 and beyond

**Date:** <!-- YYYY-MM-DD -->

**Status:** 

## Decision

As part of the development cycle for v5 we will publish one or more pre-release versions to npm.

These releases will follow our usual release process except that:

- the version number will be suffixed to indicate that it is a pre-release, in line with point 9 of [Semantic Versioning 2.0.0].
- the release will be published to npm using the `next` tag rather than the `latest` tag
- the release will be marked as a pre-release when publishing to GitHub

The suffix for the version number will initially be `-alpha.0`, for example `v5.0.0-alpha.0`. We will increment the alpha number for each pre-release.

We will consider using `beta` if we believe there is a meaningful difference in the stability of the pre-release that we want to communicate to users – in which case the incrementing number would be reset to 0.

For example:

```
v5.0.0-beta.0@next
v5.0.0-alpha.2
v5.0.0-alpha.1
v5.0.0-alpha.0
```

## Rationale

Why are we publishing to npm?

Follow semver

Unsure at this stage if we need to differentiate between different types of pre-release, using alpha gives us option of introducing others later.

next tag is sufficient, don't see why users would differentiate between alpha or beta if want to test pre-release of next version

## Risks and constraints

Users might install alpha versions in production.

## Alternatives considered

Pre-release to GitHub

## Implications

### Structure of the release notes



### Renaming pre-releases to previews

We have an existing concept which we have previously called pre-releases, which allows us to test a potential change without needing to publish it to npm. This is where we:

- create a filtered branch that contains only the contents of the package at its root
- push the branch to GitHub
- use `npm install <git remote url>` to use that version of the package elsewhere, for example in the Design System to preview documentation

Although this has been used in the past to test releases, its primary use is to test work earlier on in the development cycle, often from feature branches.

As this meets a different need, it remains useful and we should keep the processes and documentation. However, to avoid confusion, we should stop referring to these branches as 'pre-releases' and update our documentation to refer to them as 'preview branches' instead.

There is a separate issue with how preview branches are versioned which is considered out of scope for this decision record.

## Contributors

<!-- Who made this decision. -->

## Associated issues and pull requests (PRs)

<!-- Add links to any related GitHub issues and / or PRs. -->

## Outcomes

- Update docs for publishing a release to include information on what to do when publishing a pre-release
- 
