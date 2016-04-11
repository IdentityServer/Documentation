---
layout: docs-default
---

# Endpoints

The WS-Federation plugin uses the `/wsfed` URL within IdentityServer. This root site should then be used as the authority in any relying parties if you are not using the metadata endpoint and can be changed using the `MapPath` property in the `WsFederationPluginOptions`.

## Sign-in/out

Signing in and out is then built on top of this URL.

Supported parameters:

* `wa`
    * must be either `wsignin1.0` for signing in, or `wsignout1.0` for signing out
* `wtrealm`
    * realm of the relying party
* `wctx`
    * context to be round tripped back to the relying party (similar to `state` in OAuth2)
* `whr`
    * name of the external identity provider to use (skips the selection screen)

Example (encoding removed for readability):

```
/wsfed?wa=wsignin1.0&wtrealm=rp1&whr=Google
```

## Metadata

Returns the metadata document:

```
/wsfed/metadata
```