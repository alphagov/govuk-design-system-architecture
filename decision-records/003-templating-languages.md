# Templating languages

**Date:** 2018-07-05 (Historical)

**Status:** Accepted

## Context

Before GOV.UK Frontend, you needed to use 3 different libraries to build a
service that looked like GOV.UK:

- GOV.UK Frontend Toolkit
- GOV.UK Template
- GOV.UK Elements

GOV.UK Template provides the boilerplate, header, footer and related assets like
the favicon, and is available in a number of different formats:

- a gem containing a Rails engine
- a tarball containing Play Framework templates
- a folder containing Mustache templates
- a tarball containing Liquid templates
- a tarball containing Mustache Inheritance templates
- a tarball containing Jinja templates
- a tarball containing plain HTML and assets
- a tarball containing Embedded JavaScript (EJS) templates
- a JAR file containing assets but no templates, structured as per WebJars
- a tarball containing Django templates

Making GOV.UK Template available in a number of formats provides a convenient
way for service teams across government to start a new GOV.UK project, despite
the number of different tech-stacks being used.

These different formats are automatically generated from a base ruby ERB
template through the use of different '[compilers][template-compilers]',
'[packagers][template-packagers]' and '[publishers][template-publishers]' for
each target format.

GOV.UK Elements, which contains the rest of the UI Elements that you would need
to use, does not support any sort of templating language. The only thing that it
provides is an [npm package containing the Sass][elements-sass]. It relies on
users copying and pasting HTML from the examples in the online documentation.

The way that the different formats in GOV.UK Template are generated is fairly
simple. It supports the substitution of blocks and variables, but is unable to
handle loops, complex logic or many of the other features provided by most
templating languages. For this reason, we do not think that we could use the
same technique to provide the components in GOV.UK Frontend in multiple formats.
We investigated other ways of doing this in alpha, which are outlined below, but
found that doing so in a reliable way would involve significant work.

Overall we do not think that the pain points involved in maintaining a setup of
this nature are worth the benefit if we are only able to provide the 'header,
footer and boilerplate in multiple formats,  given how rarely they change. The
[HTML in GOV.UK Template has not changed very much][template-changes] in the
years it has existed.


### Pain points

- The team do not have working knowledge of a number of the target languages and
  frameworks. This would make it difficult to fix issues that might be reported,
  or to update the template to work with new versions of the different
  frameworks that might be released.

- There are integration tests for some of the target languages and frameworks,
  but not all. This makes it difficult to make substantial changes to Template
  whilst being confident that those changes will work everywhere unless you do
  substantial manual testing. It would be possible to set up test harnesses for
  every target language and framework, but this would be significant work.

- The build pipeline is reasonably complex with a number of downstream builds
  being kicked off in other repositories, making it hard to diagnose when issues
  arise. It also publishes to a number of different package managers using a
  variety of different methods.

### Transpiling templates

[Transpilation][transpilation] is the act of taking source code written in one
language and producing an equivalent version in another language.

In [alpha][alpha] the team explored ways of doing this by using [shawnbot/meta-
template][meta-template] which converts the source template into an [abstract
syntax tree][ast] and uses that to generate the destination template. We created
a [working example][alpha-transpilation] which involved  [adding an ERB
formatter][meta-template-pr] to meta-template.

However, meta-template was in its infancy and not something we could adopt
without doing significant work. It was also difficult to establish which
templating language features, for example, default variables for variables, were
going to be available in every templating language we wanted or might want to
support, and therefore which features were safe to use in the source template.

There is also the risk of making contribution more difficult, as contributors
may discover issues with transpilation during development that they do not know
how to solve. For example, tests may fail because they're using a new feature
that either doesn't work in another templating language, or that the transpiler
doesn't know how to transpile.


## References

There's a [talk by Alice Bartlett from Patterns Day 2017][alice-talk] that
touches on some of these challenges (the most relevant section starts at
around 10 minutes in).


## Decision

We will use a single templating language which will allow us to define the
markup for a component in a single place.

We will not build integrations into other languages or frameworks. Our
'deliverable' will be in the form of one or more npm packages containing the
Sass and JavaScript.

We may decide to look at transpilation again in the future, based on the
feedback that we get from our users.

## Consequences

We think there is still going to be a need for either libraries or example apps 
to help users integrate GOV.UK Frontend into their project, but we hope that
they can:

- be built and supported by teams and individuals within the community who have
  a good working knowledge of the framework or language in question.
- be built in such a way that they do not vendor GOV.UK Frontend, so that they
  do not need to be updated every time GOV.UK Frontend is updated.


[template-compilers]: https://github.com/alphagov/govuk_template/tree/master/build_tools/compiler
[template-packagers]: https://github.com/alphagov/govuk_template/tree/master/build_tools/packager
[template-publishers]: https://github.com/alphagov/govuk_template/tree/master/build_tools/publisher
[elements-sass]: https://www.npmjs.com/package/govuk-elements-sass
[template-changes]: https://github.com/alphagov/govuk_template/commits/master/source/views/layouts/govuk_template.html.erb
[transpilation]: https://en.wikipedia.org/wiki/Source-to-source_compiler
[ast]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
[meta-template]: https://github.com/shawnbot/meta-template
[meta-template-pr]: https://github.com/shawnbot/meta-template/pull/6
[alpha]: https://github.com/alphagov/govuk_frontend_alpha
[alpha-transpilation]: https://github.com/alphagov/govuk_frontend_alpha/tree/master/lib/transpilation
[alice-talk]: https://vimeo.com/226575101


