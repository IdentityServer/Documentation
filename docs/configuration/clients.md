---
layout: docs-default
---

#Clients

The `Client` class models an OpenID Connect or OAuth2 client - e.g. a native application, a web application or a JS-based application ([link](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source/Core/Models/Client.cs)).

* `Enabled`
    * Specifies if client is enabled (defaults to `false`)
* `ClientId`
    * Unique ID of the client
* `ClientSecrets`
    * List of Client secrets - only relevant for flows that require a secret
* `ClientName`
    * Client display name (used for logging and consent screen)
* `ClientUri`
    * URI to further information about client (used on consent screen)
* `LogoUri`
    * URI to client logo (used on consent screen)
* `RequireConsent`
    * Specifies whether a consent screen is required (defaults to `false`)
* `AllowRememberConsent`
    * Specifies whether user can choose to store consent decisions (defaults to `false`)
* `Flow`
    * Specifies allowed flow for client (either `AuthorizationCode`, `Implicit`, `Hybrid`, `ResourceOwner`, `ClientCredentials` or `Custom`). Defaults to `Implicit`.
* `RedirectUris`
    * Specifies allowed URIs to return tokens or authorization codes to
* `PostLogoutRedirectUris`
    * Specifies allowed URIs to redirect to after logout
* `ScopeRestrictions`
    * Specifies the scopes that the client is allowed to request. If empty, the client can request all scopes (defaults to empty)
* `IdentityTokenLifetime`
    * Lifetime to identity token in seconds (defaults to 300 seconds / 5 minutes)
* `AccessTokenLifetime`
    * Lifetime of access token in seconds (defaults to 3600 seconds / 1 hour)
* `AuthorizationCodeLifetime`
    * Lifetime of authorization code in seconds (defaults to 300 seconds / 5 minutes)
* `AccessTokenType`
    * Specifies whether the access token is a reference token or a self contained JWT token (defaults to `Jwt`).
* `AllowLocalLogin`
    * Specifies if this client can use local accounts, or external IdPs only
* `IdentityProviderRestrictions`
    * Specifies which external IdPs can be used with this client (if list is empty all IdPs are allowed). Defaults to empty.
* `CustomGrantTypeRestrictions`
    * Specifies which custom grant types the client can use (if the flow is set to `Custom`).

In addition there are a number of settings controlling the behavior of refresh tokens - see [here](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/wiki/Refresh-Tokens)