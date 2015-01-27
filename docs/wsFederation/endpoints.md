---
layout: docs-default
---

# Endpoints

The WS-Federation plugin consists of two endpoint

## Sign-in/out

That's the main WS-Federation endpoint of signing in and out.

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