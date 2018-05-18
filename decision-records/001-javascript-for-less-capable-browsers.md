# JavaScript for less-capable browsers

**Date:** 2018-05-16

**Status:** Accepted

## Context

Before GOV.UK Frontend, our projects used jQuery for DOM interactions, events and data manipulation.

We’re taking a step back from jQuery due to its lack of support for the browsers we support, its large file size, lack of security updates and from conversations with the community.

## Decision

We’re now writing standard ES5 JavaScript instead, that we polyfill where necessary.

This means that in places where we would have previously used [`$.each`](http://api.jquery.com/jquery.each/) we’re using [`.forEach`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) instead, and polyfilling the missing gaps.

We use polyfills provided by the Financial Times’ [Polyfill service](https://polyfill.io).

This approach ensures that multiple polyfills can be sourced from this service with greater confidence that they’ll work without conflicting with each other.

The Polyfill service does not do runtime detection in browsers and instead opts to do this on the server via user-agent sniffing. It only ships the code needed for that browser, which means newer browsers don’t have to run anything. We may investigate lazy-loading in the future, but for now we’re using a bundled approach based on the lowest common denominator.

We are vendoring these polyfills to avoid any [single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure) issues that could arise from relying on a CDN. By doing this, we can detect if polyfills are needed at runtime, which results in all browsers getting the same polyfill bundle.

We hope that our approach can be automated or moved into a reusable npm package, based on the Financial Times [npm package](https://github.com/Financial-Times/polyfill-service#library).

Here is an [example of polyfilling `addEventListener`](https://github.com/alphagov/govuk-frontend/blob/master/docs/polyfilling.md).

Any polyfills included in GOV.UK Frontend will be tested to work in supported browsers and devices, including assistive technology. Any community discussion and documentation around potential bugs or downsides will also be considered before deciding to include a polyfill in GOV.UK Frontend.

## Consequences

### Advantages

- We’re not forcing projects to include and rely on a specific library
- Developers don’t need to know or learn a specific library
- The JavaScript code is standard and transferable
- It’s very easy to remove the polyfills when older browser support is dropped while keeping the same JavaScript code
- Approach can be easily maintained and used by the community
- Polyfills are widely maintained by the industry

### Disadvantages
- There are risks modifying globals or native object prototypes
  - https://www.w3.org/2001/tag/doc/polyfills/#always-encapsulate
  - http://perfectionkills.com/whats-wrong-with-extending-the-dom/
- We cannot be sure if our polyfills work with other third party code

## References
- [Polyfills and the evolution of the Web](https://www.w3.org/2001/tag/doc/polyfills/)
- [Polyfill approach discussion on GitHub](https://github.com/alphagov/govuk-frontend/issues/676)
