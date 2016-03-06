---
layout: docs-default
---

# Proof of Possession - Overview (added in v2.5)

The OAuth 2.0 bearer token specification, as defined in [RFC 6750](https://tools.ietf.org/html/rfc6750),
allows any party in possession of a bearer token
(a "bearer") to get access to the associated resources (without demonstrating possession
of a cryptographic key).  To prevent misuse, bearer tokens must be
protected from disclosure in transit and at rest.

Some scenarios demand additional security protection whereby a client
needs to demonstrate possession of cryptographic keying material when
accessing a protected resource.

Proof of possesion (PoP from now on) provides a mechanism to bind key material to access tokens. This key material can then
be used by the client to apply a signature to outgoing HTTP requests to the resource server. The resource server can use the same
key material to make sure that the sender is the same entity that requested the token in the first place.

You can find the details of the PoP mechanism as well as the corresponding protocol messages
[here](https://tools.ietf.org/wg/oauth/draft-ietf-oauth-pop-architecture/) and
[here](https://tools.ietf.org/wg/oauth/draft-ietf-oauth-pop-key-distribution/).

**Note** The above specs are not done yet and we only provide an experimental implementation for asymmetric keys.
If the specs change, we also need to change the implementation. This will not be considered a breaking change.