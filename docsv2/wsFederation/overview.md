---
layout: docs-default
---

# Adding WS-Federation Support

IdentityServer has support for the WS-Federation support for both acting as an identity provider as well
as allowing external authentication over the WS-Federation protocol. For external authentication see [here](../configuration/identityProviders.html).

This section is about adding WS-Federation identity provider capabilities to IdentityServer3.

WS-Federation support is a plugin for IdentityServer3 and is implemented using the .NET 4.5 System.IdentityModel.Service assembly.
You first need to install the plugin using Nuget:

 ```
 install-package Thinktecture.IdentityServer3.WsFederation
 ```

 You can then wire up the plugin by implementing the `PluginConfiguration` callback on the `IdentityServerOptions` class like this:

 ```csharp
 public void Configuration(IAppBuilder appBuilder)
 {
     var options = new IdentityServerOptions
     {
         SiteName = "Thinktecture IdentityServer3 with WsFed",

         SigningCertificate = Certificate.Get(),
         Factory = factory,
         PluginConfiguration = ConfigureWsFederation
     };

     appBuilder.UseIdentityServer(options);
 }

 private void ConfigureWsFederation(IAppBuilder pluginApp, IdentityServerOptions options)
 {
     var factory = new WsFederationServiceFactory(options.Factory);

     // data sources for in-memory services
     factory.Register(new Registration<IEnumerable<RelyingParty>>(RelyingParties.Get()));
     factory.RelyingPartyService = new Registration<IRelyingPartyService>(typeof(InMemoryRelyingPartyService));

     var wsFedOptions = new WsFederationPluginOptions
     {
         IdentityServerOptions = options,
         Factory = factory
     };

     pluginApp.UseWsFederationPlugin(wsFedOptions);
 }
 ```

The equivalent to an OpenID Connect or OAuth2 client is called relying party in WS-Federation.
Similar to the other in-memory factories (see [here](../configuration/inMemory.html)) the WS-Federation plugin has built-in
support for retrieving relying parties from an in-memory service.

See [here](relyingParties.html) for information on how to define a relying party.
