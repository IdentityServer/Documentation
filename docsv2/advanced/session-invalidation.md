---
layout: docs-default
---

#Authentication Session Invalidation (added in v2.4)

IdentityServer3 defines the `IAuthenticationSessionValidator` interface to allow invalidating an existing login session. 
This can be used to, in essence, ignore a logged in user's authentication cookie (typically due to some external event such as the user having changed their password since they logged in).
The user will be treated as anonymous, which generally means that they must re-authenticate to continue to use IdentityServer.

## IAuthenticationSessionValidator
The interface defines one method:

* `IsAuthenticationSessionValidAsync`
 * This method is called whenever an authentication cookie is presented to IdentityServer for the logged in user. Return `true` to indicate the authentication cookie should be honored, `false` otherwise.
 * The `ClaimsPrincipal` representing the authenticated user is passed as a parameter.
