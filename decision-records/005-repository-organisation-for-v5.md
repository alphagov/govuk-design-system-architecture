# Restructure govuk-frontend repository for v5

**Date:** 2023-04-21

**Status:** Accepted

## Decision

<!-- An overview of the decision made. -->

To prepare the codebase for v5, we've agreed to make the following changes to the files in our repository and their location:

1. Create `govuk-frontend` and `govuk-frontend-review` packages
     - Create a `govuk-frontend` npm package.
          - Move source files (currently `src` ) and built package files (currently `package`) to this package.
          - Keep shipping only the built files to npm for now. We'll use the `files` field of `package.json` to include directories in the shipped package, and `.npmignore` to exclude specific files (like tests)
     - Create a `govuk-frontend-review` package for the review app
     - Store these in a `packages` folder
2. Create shared `config`, `lib`, `tasks` and `helpers` packages
3. Remove the built package files (currently `package`) from version control
4. Update our release process to make sure we keep checking the changes (possibly manually) between the package to be released and the previous version
5. Leave `dist` in its current location for now (we may revisit whether it's in version control or at the root in the future). These are compiled files for use without npm.

This will yield the following structure for our repository:

```
govuk-frontend                // Repository root
├── ...
├── dist                      // Built files for use without npm
├── packages                  // Parent folder for packages
│   ├── govuk-frontend        // The GOV.UK Frontend package
│   │   ├── dist              // [Previously `/package`] Generated build files, not in version control, ships to npm
│   │   ├── src               // [Previously `/src`] Source files, in version control, not shipped to npm
│   │   ├── tasks             // Build, release and Gulp tasks for the GOV.UK Frontend Package 
│   │   ├── package.json      // Ensures dependencies of src, dist and tasks are in-sync
│   │   └── ...
│   ├── govuk-frontend-review // The Review App package
│   │   ├── dist              // Not in version control. Some tasks need the review app to build (tests, for example)
│   │   ├── src               // [Previously `app/src`] Source files for review app
│   │   ├── tasks             // Build and watch tasks for the review app
│   │   ├── package.json
│   │   └── ...
├── shared                    // Parent folder for packages shared across the monorepo 
│   ├── config                // [Previously `/config`] Stores config info like filepaths and ports
│   ├── helpers               // Helper functions for development
│   ├── lib                   // [Previously `/lib`] Shared libraries
│   └── tasks                 // [Previously `/tasks`] Shared tasks
├── src                       // Contains symlinks to new source file locations
└── package.json              // Workspaces configuration and top-level dependencies
```

## Rationale

<!-- The reason for the decision and any comments or disagreements cited. -->

### Creating `govuk-frontend` and `govuk-frontend-review` packages

#### **Overview**
These are the packages we "ship" in some form (we publish `govuk-frontend` to npm, and we deploy `govuk-frontend-review` to Heroku), so it makes sense to group them, and the `packages` folder is a convention seen across other repos. This also allows us to better organise our dependencies between the two.

#### **`govuk-frontend`**
We may need to ship with dependencies in the future. For example:

- peer dependencies to a polyfill library if we shipped a version compiled with `import` statements for the polyfills (rather than having the polyfills bundled in)
- dependencies to other libraries, if we were shipping a Babel preset or PostCSS configuration

Moving the source and built files within the same package helps us ensure these dependencies are resolved consistently.

While we don't yet have any dependencies, moving these files changes the shape of our package, requiring users to update their import paths for Nunjucks, making it a breaking change. Taking advantage of the breaking nature of the v5 release sets us up so we don't have to worry about future dependencies.

This doesn't force us to ship the source files, which we've decided to exclude (at least for now). We can choose which folders to include via the `files` field in `package.json`, and use `.npmignore` to exclude specific files.

#### **`govuk-frontend-review`**
The review app uses relative paths (`../../src/...`) to access the components of GOV.UK Frontend. While we cannot make it consume `govuk-frontend` the exact same way our users do (downloading from an `npm` registry), we can get pretty close using workspaces. Making it use `govuk-frontend` directly helps ensure that:
- we're consuming the built files, not rebuilding from source with a process that we have to keep in sync with how the `govuk-frontend` package is built
- the actual files work as intended
- `entries` in `package.json` are properly configured
- dependencies get resolved as expected (we'll still need a different route to test `peerDependency` if we introduce them, though)

### Creating config, helpers, lib and tasks packages

These are all internal tooling packages which we don't ship, which are used across our shipped packages. Grouping them in a `shared` folder is a convenience; the main benefit of splitting them this way is dependency organisation.

### Removing built package files from version control

Since we will be consuming `govuk-frontend` from the `govuk-frontend-review` app, we need to rebuild the `govuk-frontend` package regularly, rather than only at the point of release.

If we were to leave these built files in version control, we'd risk committing built files that are somewhere between the previous release and the next one. Removing them ensures we can rebuild at will.

### Updating our release process

Including the built files in version control serves as a way to check that there's no weird code sneaking in. Changes to the package get highlighted during the review of the pull request raised to commit the new build of the package and the release.

We'll need to keep checking for unexplained changes to the package as we release, even if it is by manually diffing for a while (before further automation).

### Leaving `dist` in its current location

`dist` doesn't need to move to enable the previous points, and with the amount of changes in v5, we'll leave it out of the equation for now.

## Risks and constraints

<!-- Any accepted risks associated with the decision. -->

- There may be some named-based confusion.
     - We previously used `package` as folder to build into, rather than manually edit files. Now, the `packages` folder contains sources.
     - Similarly, we'll have a `dist` folder at root, as well as in each package's folder. This will be mitigated somewhat by removing the non-root `dist` folders from version control, but we might want to consider renaming the root `dist` folder to something like `release`.
- External contributors may be used to the current structure of the repository and confused by the relocation. We will help with this by leaving hints in README files and symlinks to the new locations.

## Alternatives considered

<!-- Other options discussed but decided against. -->

### Creating `govuk-frontend` and `govuk-frontend-review` packages

- Keeping the review app in the `app` folder and repurposing the `package` folder for the `govuk-frontend` package
     - This would create some naming confusion since we previously used `package` for automatically built code
     - We'd be splitting packages across the base directory, `shared` and `package`, which might be a bit confusing
- Holding off on `govuk-frontend` source files and built package files within the same package
     - This would mean waiting for the next breaking change to do it. We may need to have peer dependencies before that release

### Creating config, helpers, lib and tasks packages

- Moving the `shared` folder into the `packages` folder
     - The location of the shared folder was kept to root to distinguish it from the "shipped" packages in the `packages` folder

### Removing built package files from version control

- We could also have built the package to a separate folder that's not in version control (for ex. `govuk-frontend-local-build`) to ensure we consume the built files without impacting `package`. We'd have to duplicate efforts to maintain two `package.json` files and lose testing our dependency resolution, package exports and complicate the linting and Dependabot updates.
- Each PR could get a package deployed to the [Github npm registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-npm-registry) allowing its installation by the review app down the line. This requires extra infrastructure though, and doesn't help when running locally.

## Implications

<!-- Impacts of this decision. -->

- Users will need to update their import paths when migrating to v5, which we can document as part of the release notes/migration docs.
- Ongoing new components (Exit This Page, Tasklist) will need to have their sources relocated before they can be merged in the new code structure. We can help with moving things and can release Exit This Page from a support branch if it needs to go before v5.
- We need to keep making sure our configuration for the GOV.UK Prototype Kit gets recognised by the Kit and picks up our files as necessary. 

## Contributors

<!-- Who made this decision. -->
- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

<!-- Add links to any related GitHub issues and / or PRs. -->
- https://github.com/alphagov/govuk-frontend/issues/3291
- https://github.com/alphagov/govuk-frontend/issues/2886
- https://github.com/alphagov/govuk-frontend/issues/3547
- https://github.com/alphagov/govuk-frontend/issues/3548
- https://github.com/alphagov/govuk-frontend/pull/3498


## Outcomes

<!-- Note any outcomes or consequences of the decision . -->

Part of the discussion around this decision clarified that the review `app` does not need to take responsibility for all development processes. Performance monitoring, for example, can happily happen in its own workspace and feed the app something to serve if necessary. Same could go for testing that the different compiled versions work appropriately if/when we start shipping more than one variant of compiled files.

It was also mentioned, discussing the possibility of dependencies, that we'll need to be mindful of the implications of the dependencies we add (provenance, maintenance...).
