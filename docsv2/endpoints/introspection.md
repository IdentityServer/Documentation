---
layout: docs-default
---

# Introspection Endpoint

*Added in v2.2*

The introspection endpoint is an implementation of [RFC 7662](https://tools.ietf.org/html/rfc7662).

It can be used to validate reference tokens (or JWTs if the consumer does not have support for appropriate JWT or cryptographic libraries).

The introspection endpoint requires authentication using a scope credential
(only scopes that are contained in the access token are allowed to introspect the token).

### Example

```
POST /connect/introspect
Authorization: Basic xxxyyy

token=<token>
```

A successful response will return a status code of 200 and either an active or inactive token:

```
{
   "active": true,
   "sub": "123"
}
```

Unknown or expired tokens will be marked as inactive:

```
{
   "active": false,
}
```


An invalid request will return a 400 or a 401 if the scope is not authorized.

**Remark** The introspection endpoint replaces the older access token validation endpoint.
Since the introspection endpoint requires authentication, it adds privacy features to reference tokens, that were
not available previously.
The access token validation endpoint still exists, but it is recommended to disable it on the `EndpointOptions` and
use the introspection endpoint instead.