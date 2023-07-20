# JavaScript file names and formats

**Date:** 2023-06-27

**Status:** Accepted

## Decision

We will consolidate all published JavaScript in the [`govuk-frontend` npm package](https://www.npmjs.com/package/govuk-frontend) into a single `govuk` directory.

We will publish JavaScript in the following formats:

1. ECMAScript (ES) modules
2. ECMAScript (ES) modules, bundled
3. Universal Module Definition (UMD), bundled

We will use ECMAScript (ES) modules by default and only bundle JavaScript files that are considered "entry points" into GOV.UK Frontend, such as `all.mjs` and exported components (for example `accordion.mjs`).

We will change our [GitHub release](https://github.com/alphagov/govuk-frontend/releases) JavaScript `govuk-frontend-${version}.min.js` to ECMAScript (ES) modules format and add it to the published [`govuk-frontend` npm package](https://www.npmjs.com/package/govuk-frontend) without a version number.

For consistency, we will clearly update our file extensions to:

1. Add prefix `.bundle` to bundled JavaScript file extensions
2. Add prefix `.min` to minified JavaScript file extensions
3. Always use `*.mjs` for ES modules (except for our [GitHub release](https://github.com/alphagov/govuk-frontend/releases), see constraints below)

See the example directory listing below to show how files in `govuk-frontend/dist` will be named, further to our [decision to restructure `govuk-frontend`](./005-repository-organisation-for-v5.md):

```shell
govuk-frontend/dist
└── govuk
    ├── govuk-frontend.min.js         # ECMAScript (ES) module bundle, minified
    │
    ├── all.mjs                       # ECMAScript (ES) module
    ├── all.bundle.mjs                # ECMAScript (ES) module bundle
    ├── all.bundle.js                 # Universal Module Definition (UMD) bundle
    │
    └── components
        ├── accordion
        │   ├── accordion.mjs         # ECMAScript (ES) module
        │   ├── accordion.bundle.mjs  # ECMAScript (ES) module bundle
        │   └── accordion.bundle.js   # Universal Module Definition (UMD) bundle
        │
        └── button
            ├── button.mjs            # ECMAScript (ES) module
            ├── button.bundle.mjs     # ECMAScript (ES) module bundle
            └── button.bundle.js      # Universal Module Definition (UMD) bundle
```

## Rationale

We can avoid future breaking changes by changing file names and extensions to clearly show:

* different "flavours" of JavaScript like ES modules
* which files are standalone, bundled or minified

For example, as explained in our [**JavaScript browser compatibility** decision](./006-javascript-compatibility.md) we may need to add polyfills or transpilation helpers in future.

Service users can optionally either import `*.bundle.mjs` (ES modules, bundled) or require `*.bundle.js` (UMD, bundled) to include all the polyfills they need, or alternatively import `*.mjs` (ES modules) to use their own bundler.

## Risks and constraints

### Browser Content-Type headers and `*.mjs` extensions

Due to poor web server `Content-Type` header support for `*.mjs` files, we've kept the `*.js` extension for our [GitHub release](https://github.com/alphagov/govuk-frontend/releases) JavaScript `govuk-frontend-${version}.min.js` bundle.

For reference:

* [Apache issue, resolved May 2022](https://bz.apache.org/bugzilla/show_bug.cgi?id=61383)
* [Nginx issue, unresolved](https://trac.nginx.org/nginx/ticket/2216)

### GitHub release format changed to ES modules

We've changed our [GitHub release](https://github.com/alphagov/govuk-frontend/releases) JavaScript `govuk-frontend-${version}.min.js` bundle format from Universal Module Definition (UMD) to ES modules, but there is a risk that service users will still require UMD.

### Maintaining CommonJS compatibilty

We know that CommonJS compatibility is important for both [Node.js `require('govuk-frontend')` package resolution](https://github.com/alphagov/govuk-frontend/issues/3755) and for test runners that do not support ES modules yet.

For example, by making our npm package "ESM only" we'd hit issues such as:

1. [Jest ECMAScript modules support](https://jestjs.io/docs/ecmascript-modules) needing `--experimental-vm-modules`
2. [Mocha current limitations](https://mochajs.org/#current-limitations) like [`--watch` mode](https://github.com/mochajs/mocha/issues/4374) only works with CommonJS

Due to this constraint we've maintained CommonJS support by continuing to package Universal Module Definition (UMD) bundles.

## Alternatives considered

### Using directories for JavaScript formats

We considered using directories to show different JavaScript formats but preferred file names and extensions:

```shell
govuk-frontend/dist
└── govuk
    ├── cjs
    ├── mjs
    └── umd
```

### Using file extension prefixes for JavaScript formats

We considered always using `*.js` file extensions and adding JavaScript formats as prefixes, but preferred using prefixes to clearly show standalone, bundled or minified files instead:

```shell
govuk-frontend/dist
└── govuk
    ├── all.cjs.js
    ├── all.mjs.js
    └── all.umd.js
```

### Separate CommonJS modules

We acknowledge the Node.js support constraint above prevents us from removing CommonJS support. That said, we chose not to publish separate `*.cjs` (CommonJS modules) since CommonJS support is still provided by our Universal Module Definition (UMD) bundles.

## Implications

- Users that follow our [**Add the JavaScript file to your HTML**](https://frontend.design-system.service.gov.uk/importing-css-assets-and-javascript/#add-the-javascript-file-to-your-html) or [**Importing JavaScript**](https://github.com/alphagov/govuk-frontend/blob/main/packages/govuk-frontend/README.md#importing-javascript) documentation will need to replace `govuk/all.js` with `govuk/all.bundle.js`.
- Users that directly import/require `govuk/**/*.js` UMD bundles will need to add the `.bundle` extension prefix as `govuk/**/*.bundle.js`.
- Users that directly import `govuk-esm/**/*.mjs` ES modules will need to remove the `-esm` directory suffix as `govuk/**/*.mjs`.

**Note:** Users that import/require `govuk-frontend` by name are unaffected.

## Contributors

- Brett Kyle (@domoscargin)
- Colin Rotherham (@colinrotherham)
- Oliver Byford (@36degrees)
- Romaric Pascal (@romaricpascal)

## Associated issues and pull requests (PRs)

- https://github.com/alphagov/govuk-frontend/issues/3721
- https://github.com/alphagov/govuk-frontend/pull/3726
- https://github.com/alphagov/govuk-design-system-architecture/pull/28

## Outcomes

<!-- Note any outcomes or consequences of the decision . -->
