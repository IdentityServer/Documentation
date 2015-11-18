---
layout: docs-default
---

#OWIN Environment Extensions

IdentityServer3 provides extension methods for the OWIN environment (`IDictionary<string, object>`) in the `IdentityServer3.Core.Extensions` namespace. These can be used to access features of IdentityServer from custom pages running in the same host as IdentityServer. Note that these can only be used when the custom pages are running in the same OWIN pipeline as IdentityServer.

* `GetIdentityServerHost`
    * Returns the host name of IdentityServer.
* `GetIdentityServerBasePath`
    * Returns the base path of IdentityServer in the host.
* `GetIdentityServerBaseUrl`
    * Returns the URL for IdentityServer (which combines the `GetIdentityServerHost` and `GetIdentityServerBasePath`).
* `CreateSignInRequest`
    * Creates a request to signin and returns the URL to redirect the user to for the login page.
* `GetSignInMessage`
    * Gets the current `SignInMessage`.
* `IssueLoginCookie`
    * Issues the full login cookie. Could be used instead of using the built-in authentication workflow (e.g. to log the user in from a registration page).
    * The `AuthenticatedLogin` parameter contains these properties:
        * `Subject`: The subject identifier for the user.
        * `Name`: The display name for the user.
        * `Claims`: Optional claims to issue in the login cookie.
        * `AuthenticationMethod`: Sets the `amr` claim.
        * `IdentityProvider`: Sets the `idp` claim.
        * `PersistentLogin`: Issues a persistent cookie.
        * `PersistentLoginExpiration`: Controls persistent cookie expiration.
* `GetIdentityServerLogoutUrl`
    * Returns the URL to log the user out of IdentityServer.
* `CreateSignOutRequest`
    * Creates a request to trigger single sign-out at the end session endpoint. Returns the URL to redirect the user to.
* `GetCurrentUserDisplayName`
    * Returns the current logged in user's display name.
* `GetIdentityServerFullLoginAsync`
    * Returns the `ClaimsIdentity` for the currently logged in user.
* `GetIdentityServerPartialLoginAsync`
    * Returns the `ClaimsIdentity` for the current partial login.
* `GetPartialLoginRestartUrlAsync`
    * Returns the URL to the login page for the current partial login.
* `GetPartialLoginResumeUrlAsync`
    * Returns the URL to continue logging in the user for the current partial login.
* `UpdatePartialLoginClaimsAsync`
    * Replaces the claims for the current partial login.
* `RemovePartialLoginCookie`
    * Removes the cookie for the current partial login.
* `GetRequestId`
    * Returns the current request ID used for logging.
* `ResolveDependency` (added in v2.2)
    * Resolves dependency type from the IdentityServer DI system.
* `ProcessFederatedSignoutAsync` (added in v2.2)
    * Revokes authentication cookies and renders HTML to trigger single signout of all clients. This is intended to be used within an iframe when an external, upstream IdP is providing a signout callback to IdentityServer for single signout.
* `RenderLoggedOutViewAsync` (added in v2.2)
    * Requests that the logged out view be rendered and the signout message cookie be removed.
* `GetSignOutMessageId` (added in v2.2)
    * Gets the sign out message id.
* `GetSignOutMessage` (added in v2.2)
    * Gets the sign out message.
