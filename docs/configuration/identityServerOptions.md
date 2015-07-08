---
layout: docs-default
---

#IdentityServer Options

The `IdentityServerOptions` [class](https://github.com/thinktecture/Thinktecture.IdentityServer.v3/blob/master/source%2FCore%2FConfiguration%2FIdentityServerOptions.cs) is the top level container for all configuration settings of IdentityServer.

* `IssuerUri`
    * Unique name of this server instance, e.g. https://myissuer.com. Defaults to the base URL where IdentityServer is installed
* `SiteName`
    * Display name of the site used in standard views.
* `SigningCertificate`
    * X.509 certificate (and corresponding private key) for signing security tokens.
* `SecondarySigningCertificate`
    * A secondary certificate that will appear in the discovery document. Can be used to prepare clients for certificate rollover.
* `RequireSsl`
    * Indicates if SSL is required for IdentityServer. Defaults to `true`.
* `PublicOrigin`
    * By default, IdentityServer uses the host, protocol, and port from the HTTP request when creating links.
    This might not be accurate in reverse proxy or load-balancing situations.
    You can override the origin used for link generation using this property.
* `Endpoints`
    * Allows enabling or disabling specific endpoints (by default all endpoints are enabled).
* `Factory` (required)
    * Sets the [IdentityServerFactory](serviceFactory.html)
* `DataProtector`
    * Sets a custom data protector. By default the Katana host data protector is used.
* `AuthenticationOptions`
    * Configures [AuthenticationOptions](authenticationOptions.html)
* `PluginConfiguration`
    * Allows adding protocol plugins like the WS-Federation support.
* `CorsPolicy`
    * Configures [CORS](cors.html)
* `CspOptions`
    * Configures [CSP](../advanced/csp.html)
* `ProtocolLogoutUrls`
    * Configures callback URLs that should be called during logouts (mainly useful for protocol plugins).
* `LoggingOptions`
    * Configures settings related to [logging](logging.html)
* `EventsOptions`
    * Configures settings related to [events](events.html)
* `EnableWelcomePage`
    * Enables or disables the default welcome page. Defaults to `true`.
