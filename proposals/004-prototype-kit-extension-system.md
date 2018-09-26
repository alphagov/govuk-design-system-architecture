# GOVUK Prototype Extensions

**Closing date:** 2018-10-05

**Status:** proposed

## Summary

The extension we’re proposing aims to make it easier for:

prototype users to install and use patterns and components from any government department
 - developers in government departments to make their custom components available for use in prototypes within their department
 - developers in government departments to share their custom components for use in other government departments
 - developers in government departments to extend the GOV.UK Prototype Kit
 - developers in government departments to contribute their components to GOVUK Frontend

## Problem

Designers who want to use department-specific components in their prototypes have to follow complicated steps which can be intimidating and difficult to follow. In [HMCTS’s documentation](https://github.com/hmcts/frontend/blob/7b52ef51aad9b2b5453db23e3f20aeb23b73a08e/docs/installation/installing-with-npm.md) there are a number of complicated installation steps to follow which can cause errors. We want to avoid this and think that each government department will need their own version of these steps.

## Proposal

Developers creating components for government departments could provide an npm package containing components and a configuration file specifying the integration points required to make those components available to prototype users.

An updated version of the GOV.UK Prototype Kit could support the installation of these npm packages so that designers do not have to make manual modifications of core prototype kit files.

By including a [configuration file](https://github.com/hmcts/frontend/issues/31) HMCTS and HMRC components can be installed via npm and then used immediately without prototype users needing to make changes to source files.

Govuk-frontend could be integrated in the same way, this both proves that the extension system is powerful enough to support govuk’s needs and removes hard-coded references to the govuk-frontend module.

## Proposed Implementation

Any node module which is in the package.json file containing a govuk-prototype-kit.config.json file in its root directory will be processed as an extension.

This json file can contain a number of keys, all the values are arrays.  All paths are relative to the package root.  The options are:

 - `nunjucksPaths` - directories to include in the appViews, these directories contain nunjucks includes
 - `sass` - sass files to include in application.css
 - `stylesheets` - CSS files to be included as-is into each page of the prototype
 - `scripts` - JS files to be included as-is into each page of the prototype
 - `assets` - directories or files which should be made available under a namespaced path e.g. `/extension-assets/EXTENSION_NAME/path/to/asset.txt`, this can also be used to add to the `/assets` context path used by `govuk-frontend`

The changes can be seen [in the extensions branch of HMRC’s fork](https://github.com/hmrc/govuk_prototype_kit/pull/37/files)

The configuration for govuk-frontend would be:

```JSON
{
  "nunjucksPaths": ["/", "/components"],
  "scripts": ["/all.js"],
  "assets": [{"global": true, "src": "/assets"}],
  "sass": ["/all.scss"]
}
```

A working draft of the hmrc-frontend is:

```JSON
{
  "assets": ["/src/components/account-menu/images"],
  "nunjucksPaths": ["/src/components"],
  "sass": ["/src/_prototype-kit.scss"],
  "scripts": ["/src/components/account-menu/account-menu.js"]
}
```

In most cases the order of extensions won’t be important because CSS, Javascript and Macros  are expected to be namespaced.  Extension authors should avoid defining the same CSS selectors, nunjucks macros etc. but if they do the order becomes important.  

### Specifying the order of extensions

We expect conflicts between extensions to be very rare but when they occur it would be good for the prototype user to have a solution available.  In the proposed implementation this is done by editing `foundationExtensions` in `app/config.js`.

As different technologies deal with overrides in different ways (for Nunjucks the first takes priority, in CSS the last takes priority) the extension priorities are normalised so that later extensions will always override the earlier ones.  The order will be:

First: The installed modules listed in foundationExtensions in the order specified
Then: All other npm modules containing a config file in alphabetical order

Out of the box the prototype kit defines “govuk-frontend” as the only priority extension, therefore everything the user specifically installs takes priority over govuk-frontend unless otherwise specified.

If we decided not to define the order of extensions the prototype user would experience inconsistency in their prototype in the event of conflicts, these would not be easy to diagnose or resolve.

## Fully functional example

[There is a working example of all these features available](https://github.com/yagni-digital/govuk-plugin-example)
