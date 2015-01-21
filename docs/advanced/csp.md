---
layout: docs-default
---

#CSP

IdentityServer incorporates the use of [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/Security/CSP) (CSP) for all HTML pages displayed.

### CspOptions

IdentityServer v3 allows the hosting application to configure a `CspOptions` on the `IdentityServerOptions` to control the CSP behavior. Below are the settings that are configurable:

* `Enabled` : indicates if CSP is enabled or disabled. Defaults to `true`.
* `ScriptSrc` : allows for additional `script-src` values to be added to the default policy.
* `StyleSrc` : allows for additional `style-src` values to be added to the default policy.
* `FontSrc` : allows for additional `font-src` values to be added to the default policy.
* `ConnectSrc` : allows for additional `connect-src` values to be added to the default policy.

CSP allows for a reporting endpoint to be configured. IdentityServer provides a CSP report endpoint that is enabled by default. CSP errors will be raised as [events](../configuration/events.html) in the eventing system. The CSP reporting feature can disabled by setting the `EnableCspReportEndpoint` property to `false` on the `EndpointOptions` which is a property of the [`IdentityServerOptions`](../configuration/identityServerOptions.html).
