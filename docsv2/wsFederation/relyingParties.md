---
layout: docs-default
---

# Relying Parties

The `RelyingParty` class models a WS-Federation relying party:

* `Name`
    * Display name (used for logging)
* `Enabled`
    * Specifies if relying party is enabled. Defaults to `true`
* `Realm`
    * Unique identifier of the relying party
* `ReplyUrl`
    * URL to send the token back to (maps to wreply parameter)
* `TokenType`
    * Type of the token to return. Default to SAML 2.0. The following types are supported:
        * `urn:oasis:names:tc:SAML:1.0:assertion` (SAML 1.1)
        * `urn:oasis:names:tc:SAML:2.0:assertion` (SAML 2.0)
        * `urn:ietf:params:oauth:token-type:jwt` (JWT)
* `TokenLifeTime`
    * Token lifetime in minutes (defaults to 600)
* `EncryptingCertificate`
    * Certificate for encrypting the token (SAML only). Note that this is separate to the SSL or signing certificate
* `IncludeAllClaimsForUser`
    * Includes all available claims of the user in the token (as opposed to the explicit mappings)
* `DefaultClaimTypeMappingPrefix`
    * Default prefix for the output claim type if IncludeAllClaimsForUser is set but no explicit mapping exists for the claim. Required when using SAML 1.1
* `ClaimMappings`
    * Allows setting up a mappings table from the internal claim types to outgoing claim types (e.g.for situation where you feel the urge to map from `name` to `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`)
* `SamlNameIdentifierFormat`
   * Allows setting the SAML name identifier format for SAML name identifier claims
* `SignatureAlgorithm`
   * Allows setting the signature algorithm for the token (defaults to RSASHA256)
* `DigestAlgorithm`
   * Allows setting the digest algorithm (defaults to SHA256)
