---
layout: docs-default
---

# CSP

IdentityServer incorporates the use of [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/Security/CSP) (CSP) for all HTML pages displayed.

### CspOptions

IdentityServer3 allows the hosting application to configure a `CspOptions` on the `IdentityServerOptions` to control the CSP behavior. Below are the settings that are configurable:

* `Enabled` : indicates if CSP is enabled or disabled. Defaults to `true`.
* `ScriptSrc` : allows for additional `script-src` values to be added to the default policy.
* `StyleSrc` : allows for additional `style-src` values to be added to the default policy.
* `FontSrc` : allows for additional `font-src` values to be added to the default policy.
* `ConnectSrc` : allows for additional `connect-src` values to be added to the default policy.
* `ImgSrc`: allows for additional `img-src` values to be added to the default policy.
* `FrameSrc` (added in v2.4) : allows for additional `frame-src` values to be added to the default policy.

CSP allows for a reporting endpoint to be configured. IdentityServer provides a CSP report endpoint which is described [here](../endpoints/csp.html).
