---
layout: docs-default
---

# Service Factory

The `WsFederationServiceFactory` builds upon the existing `IdentityServerServiceFactory` with both mandatory and optional services.
The UserService and therefore the user store will be the same as that registered in Identity Server.


## Mandatory

* `RelyingPartyService`
    * Implements the retrieval of relying party configuration data ([interface](https://github.com/IdentityServer/IdentityServer3.WsFederation/blob/master/source/WsFederationPlugin/Services/IRelyingPartyService.cs))

## Optional

* `CustomRequestValidator`
	* Implements custom additional validation of sign in requests ([interface](https://github.com/IdentityServer/IdentityServer3.WsFederation/blob/master/source/WsFederationPlugin/Services/ICustomWsFederationRequestValidator.cs))
* `CustomClaimsService`
	* Implements claims transformation based on both requesting user and relying party ([interface](https://github.com/IdentityServer/IdentityServer3.WsFederation/blob/dev/source/WsFederationPlugin/Services/ICustomWsFederationClaimsService.cs))
* `RedirectUriValidator`
	* Models the logic when validating post logout redirect URIs. ([interface](https://github.com/IdentityServer/IdentityServer3.WsFederation/blob/dev/source/WsFederationPlugin/Services/IRedirectUriValidator.cs))
	
