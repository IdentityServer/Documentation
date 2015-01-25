---
layout: docs-default
---

# Defining Relying Parties

The `RelyingParty` class models a relying party:

* `Name`
    * Display name
* `Enabled`
    * Enabled/disabled (defaults to `true`)
* `Realm`
    * Unique identifier of the relying party
* `ReplyUrl`
    * URL to send token back to
* `TokenType`
    * Type of the token (defaults to SAML2), the following types are supported:
        * `urn:oasis:names:tc:SAML:1.0:assertion` (SAML 1.1)
        * `urn:oasis:names:tc:SAML:2.0:assertion` (SAML 2.0)
        * `urn:ietf:params:oauth:token-type:jwt` (JWT)
* `TokenLifeTime`
    * Token lifetime in minutes (defaults to 480)
* `EncryptingCertificate`
    * Certificate for encrypting the token (SAML only)
* `IncludeAllClaimsForUser`
    * Includes all available claims of the user in the token (as opposed to the explicit mappings)
* `DefaultClaimTypeMappingPrefix`
    * Default prefix for the output claim type if `IncludeAllClaimsForUser` is set but no explicit mapping exists.
* `ClaimMappings`
    * Allows setting up a mappings table from the internal claim types to outgoing claim types (for situation where you
    feel the urge to map e.g. from `name` to `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`)

