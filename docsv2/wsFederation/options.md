---
layout: docs-default
---

# Options

The `WsFederationPluginOptions` class contains the public configuration settings of the WS-Federation plugin:

* `IdentityServerOptions`
	* The `IdentityServerOptions` passed through the `PluginConfiguration` callback as a parameter
* `Factory`
	* Sets the `WsFederationServiceFactory`
* `EnableMetadataEndpoint`
	* Enables the WS-Federation metadata endpoint. Defaults to `true`
* `MapPath`
	* The URL within IdentityServer reserved for WS-Federation. Defaults to `/wsfed`