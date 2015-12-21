---
layout: docs-default
---

#Scopes and Claims

**The `IdentityServer.Core.Models.Scope` class models an OpenID Connect or OAuth2 scope.**

* `Enabled`
    * Indicates if scope is enabled and can be requested. Defaults to `true`.
* `Name`
    * Name of the scope. This is the value a client will use to request the scope.
* `DisplayName`
    * Display name for consent screen.
* `Description`
    * Description for the consent screen.
* `Required`
    * Specifies whether the user can de-select the scope on the consent screen. Defaults to `false`.
*  `ScopeSecrets` (added in v2.2)
    * Adds secret to scope (for the introspection endpoint) - see also [here](secrets.html).
*  `AllowUnrestrictedIntrospection` (adden in v2.3)
    * Allows this scope to see all other scopes in the access token when using the introspection endpoint
* `Emphasize`
    * Specifies whether the consent screen will emphasize this scope. Use this setting for sensitive or important scopes. Defaults to `false`.
* `Type`
    * Either `Identity` (OpenID Connect related) or `Resource` (OAuth2 resources). Defaults to `Resource`.
* `Claims`
    * List of user claims that should be included in the identity (identity scope) or access token (resource scope). 
* `IncludeAllClaimsForUser`
    * If enabled, all claims for the user will be included in the token. Defaults to `false`.
* `ClaimsRule`
    * Rule for determining which claims should be included in the token (this is implementation specific)
* `ShowInDiscoveryDocument`
    * Specifies whether this scope is shown in the discovery document. Defaults to `true`.

**Scope can also specify claims that go into the corresponding token - the `ScopeClaim` class has the following properties:**

* `Name`
    * Name of the claim
* `Description`
    * Description of the claim
* `AlwaysIncludeInIdToken`
    * Specifies whether this claim should always be present in the identity token (even if an access token has been requested as well). Applies to identity scopes only. Defaults to `false`.

**Example of a `role` identity scope:**

```csharp
var roleScope = new Scope
{
    Name = "roles",
    DisplayName = "Roles",
    Description = "Your organizational roles",
    Type = ScopeType.Identity,

    Claims = new List<ScopeClaim>
    {
        new ScopeClaim(Constants.ClaimTypes.Role, alwaysInclude: true)
    }
};
```
The 'AlwaysIncludeInIdentityToken' property specifies that a certain claim should always be part of the identity token, 
even when an access token for the userinfo endpoint is requested.

**Example of a scope for the `IdentityManager` API:**

```csharp
var idMgrScope = new Scope
{
    Name = "idmgr",
    DisplayName = "IdentityManager",
    Type = ScopeType.Resource,
    Emphasize = true,

    Claims = new List<ScopeClaim>
    {
        new ScopeClaim(Constants.ClaimTypes.Name),
        new ScopeClaim(Constants.ClaimTypes.Role)
    }
};
```
