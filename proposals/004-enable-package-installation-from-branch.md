# Enable package installation from branch

**Closing date:** 2018-11-02

**Status:** proposed

## Summary

Enable consumption of GOV.UK Frontend from a repository branch by refactoring
the folder structure and updating the automated scripts.

Example of npm installation instructions from a branch:

`npm install --save alphagov/govuk-frontend#branch-name`

## Problem

Currently it is not possible to consume an unreleased version of GOV.UK Frontend.

There are three main reasons for wanting to do that:
1. You want to integrate a component that is work in progress into your project.
This is useful for getting early feedback on a component through user research
especially when a project is in early stages (i.e. alpha or private beta)

1. You want to prepare guidance and examples in the Design System for a component
that is in the process of contribution.

1. You want to test an unreleased version of GOV.UK Frontend in the Design System.
This is particularly useful to make sure the Design System works as expected after
major GOV.UK frontend releases.

## Proposal

In order for a package to be consumed from a repository branch, the `package.json`
for the published package MUST sit at the root level. This conflicts with the
current model in which the `package.json` for the preview app is the root level.

One way to overcome this is to move the `package.json` for the preview app at
the `app` folder level and update the script paths to reflect the change. Then
move the `package.json` for the published package at the root level and configure
the automated scripts to only publish the `package` folder.

Current directory structure of the `govuk-frontend` repository (simplified to only show folders and `package.json` files)

```bash
.
├── app
│   ├── assets
│   └── views
├── bin
├── config
├── dist
├── docs
├── lib
├── package.json # review app dependencies
├── package
│   ├── assets
│   ├── components
│   ├── core
│   ├── helpers
│   ├── objects
│   ├── overrides
│   ├── package.json # published package dependencies
│   ├── settings
│   ├── tools
│   ├── utilities
│   └── vendor
├── src
└── tasks
```

Proposed directory structure of the `govuk-frontend` repository (simplified to only show folders and `package.json` files)
```bash
.
├── app
│   ├── assets
│   ├── package.json # review app dependencies
│   └── views
├── bin
├── config
├── dist
├── docs
├── lib
├── package.json # published package dependencies
├── package
│   ├── assets
│   ├── components
│   ├── core
│   ├── helpers
│   ├── objects
│   ├── overrides
│   ├── settings
│   ├── tools
│   ├── utilities
│   └── vendor
├── src
└── tasks
```

### References
- [npm-install](https://docs.npmjs.com/cli/install)
