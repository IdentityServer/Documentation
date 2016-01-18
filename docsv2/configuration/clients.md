---
layout: docs-default
---

#Clients

The `Client` class models an OpenID Connect or OAuth2 client - e.g. a native application, a web application or a JS-based application ([link](https://github.com/IdentityServer/IdentityServer3/blob/master/source/Core/Models/Client.cs)).

* `Enabled`
    * Specifies if client is enabled. Defaults to `true`.
* `ClientId`
    * Unique ID of the client
* `ClientSecrets`
    * List of client secrets - only relevant for flows that require a secret
* `ClientName`
    * Client display name (used for logging and consent screen)
* `ClientUri`
    * URI to further information about client (used on consent screen)
* `LogoUri`
    * URI to client logo (used on consent screen)
* `RequireConsent`
    * Specifies whether a consent screen is required. Defaults to `true`.
* `AllowRememberConsent`
    * Specifies whether user can choose to store consent decisions. Defaults to `true`.
* `Flow`
    * Specifies allowed flow for client (either `AuthorizationCode`, `Implicit`, `Hybrid`, `ResourceOwner`, `ClientCredentials` or `Custom`). Defaults to `Implicit`.
* `AllowClientCredentialsOnly `
    * Gets or sets a value indicating whether this client is allowed to request token using client credentials only. This is e.g. useful when you want a client to be able to use both a user-centric flow like implicit and additionally client credentials flow. Defaults to false. Should only be used for confidential clients (e.g. not Implicit).
* `RedirectUris`
    * Specifies the allowed URIs to return tokens or authorization codes to
* `PostLogoutRedirectUris`
    * Specifies allowed URIs to redirect to after logout
* `LogoutUri` (added in v2.2)
    * Specifies logout URI at client for HTTP based logout
* `LogoutSessionRequired` (added in v2.2)
    * Specifies is the user's session id should be sent to the LogoutUri. Defaults to true.
* `RequireSignOutPrompt` (added in v2.4)
    * Specifies if the client will always show a confirmation page for sign-out. Defaults to false.
* `AllowedScopes`
    * By default a client has no access to any scopes - either specify the scopes explicitly here (recommended) - 
      or set `AllowAccessToAllScopes` to true.
* `AllowedCustomGrantTypes`
      * When `Custom` flow is used, you also need to specify which custom grant types this client can use.
        Explicitly specify the grant types here (recommended) or set `AllowAccessToAllCustomGrantTypes` to true.
* `IdentityTokenLifetime`
    * Lifetime to identity token in seconds (defaults to 300 seconds / 5 minutes)
* `AccessTokenLifetime`
    * Lifetime of access token in seconds (defaults to 3600 seconds / 1 hour)
* `AuthorizationCodeLifetime`
    * Lifetime of authorization code in seconds (defaults to 300 seconds / 5 minutes)
* `AbsoluteRefreshTokenLifetime`
    * Maximum lifetime of a refresh token in seconds. Defaults to 2592000 seconds / 30 days
* `SlidingRefreshTokenLifetime`
    * Sliding lifetime of a refresh token in seconds. Defaults to 1296000 seconds / 15 days
* `RefreshTokenUsage`
    * `ReUse`: the refresh token handle will stay the same when refreshing tokens
    * `OneTime`: the refresh token handle will be updated when refreshing tokens
* `RefreshTokenExpiration`
    * `Absolute`: the refresh token will expire on a fixed point in time (specified by the AbsoluteRefreshTokenLifetime)
    * `Sliding`: when refreshing the token, the lifetime of the refresh token will be renewed (by the amount specified in SlidingRefreshTokenLifetime). The lifetime will not exceed `AbsoluteRefreshTokenLifetime`.
* `AccessTokenType`
    * Specifies whether the access token is a reference token or a self contained JWT token (defaults to `Jwt`).
* `EnableLocalLogin`
    * Specifies if this client can use local accounts, or external IdPs only. Defaults to `true`.
* `IdentityProviderRestrictions`
    * Specifies which external IdPs can be used with this client (if list is empty all IdPs are allowed). Defaults to empty.
* `IncludeJwtId`
    * Specifies whether JWT access tokens should have an embedded unique ID (via the `jti` claim).
* `AllowedCorsOrigins`
    * If specified, will be used by the default CORS policy service implementations (In-Memory and EF) to build a CORS
      policy for JavaScript clients.

* `Claims`
    * Allows settings claims for the client (will be included in the access token).
* `AlwaysSendClientClaims`
    * If set, the client claims will be send for every flow. If not, only for client credentials flow (default is `false`)
* `PrefixClientClaims`
    * If set, all client claims will be prefixed with `client_` to make sure they don't accidentally collide with user claims. Default is `true`.

In addition there are a number of settings controlling the behavior of refresh tokens - see [here](../advanced/refreshTokens.html)

##Example: Configure a client for implicit flow

```csharp
var client = new Client
{
    ClientName = "JS Client",
    Enabled = true,

    ClientId = "implicitclient",
    Flow = Flows.Implicit,

    RequireConsent = true,
    AllowRememberConsent = true,

    RedirectUris = new List<string>
    {
        "https://myapp/callback.html",
    },

    PostLogoutRedirectUris = new List<string>
    {
        "http://localhost:23453/index.html",
    }
}
```

##Example: Configure a client for resource owner flow

```csharp
var client = new Client
{
    ClientName = "Legacy Client",
    Enabled = true,

    ClientId = "legacy",
    ClientSecrets = new List<Secret>
    {
        new Secret("4C701024-0770-4794-B93D-52B5EB6487A0".Sha256())
    },

    Flow = Flows.ResourceOwner,

    AbsoluteRefreshTokenLifetime = 86400,
    SlidingRefreshTokenLifetime = 43200,
    RefreshTokenUsage = TokenUsage.OneTimeOnly,
    RefreshTokenExpiration = TokenExpiration.Sliding
}
```
