---
layout: docs-default
---

# Overview

IdentityServer3 supports the WS-Federation protocol for both acting as an identity provider and consuming external identity providers. 

For integrating IdentityServer with external WS-Federation identity providers, such as ADFS, see the [Identity Providers section of the documentation](../configuration/identityProviders.html). 
This area of the documentation covers how to add WS-Federation identity provider capabilities to IdentityServer3.

## Installation

WS-Federation support is a plugin for IdentityServer3 and is implemented using the .NET 4.5 System.IdentityModel.Service assembly. 
You first need to install the plugin using Nuget:

 ```
 install-package IdentityServer3.WsFederation
 ```

 The plugin is wired into IdentityServer by implementing the `PluginConfiguration` callback in the `IdentityServerOptions` class:
 
```csharp
public void Configuration(IAppBuilder appBuilder)
{
    var options = new IdentityServerOptions
    {
        SiteName = "IdentityServer3 with WsFed",

        SigningCertificate = Certificate.Get(),
        Factory = factory,
        PluginConfiguration = ConfigureWsFederation
    };

    appBuilder.UseIdentityServer(options);
}

private void ConfigureWsFederation(IAppBuilder pluginApp, IdentityServerOptions options)
{
    var factory = new WsFederationServiceFactory(options.Factory);
	factory.UseInMemoryRelyingParties(RelyingParties.Get());

    var wsFedOptions = new WsFederationPluginOptions
    {
        IdentityServerOptions = options,
        Factory = factory
    };

    pluginApp.UseWsFederationPlugin(wsFedOptions);
}
```

The WS-Federation plugin uses its own ServiceFactory for registering services.
In this case we are registering a list of relying parties and an in-memory implementation of the required `IRelyingPartyService` (similar to the [other in-memory services and stores](../configuration/inMemory.html)).
`IRelyingPartyService` is the only mandatory registration.

A relying party is the WS-Federation equivalent of an OpenId Connect or OAuth2 client. 
See [here](relyingParties.html) for information on how to define a relying party.
