---
layout: docs-default
---

#Overview

IdentityServer3 is packaged as middleware and uses the typical "Options" patterns for configuration:

```csharp
public void Configuration(IAppBuilder appBuilder)
{
    var options = new IdentityServerOptions
    {
        SigningCertificate = Certificate.Get(),
        Factory = factory,
    };

    appBuilder.UseIdentityServer(options);
}
```

The `IdentityServerOptions` class contains all configuration for IdentityServer.
One part consists of simple properties like the issuer name or site title which you can source from wherever you see fit (static in code, configuration file or database).
The other part is the so called service factory which acts as a registry for certain aspects of IdentityServer's internal processing.

#### Hosting in IIS and RAMMFAR

The files for the web pages are served up as embedded assets within the IdentityServer assembly itself. When hosting in IIS or IIS Express to allow these files to be served RAMMFAR (_runAllManagedModulesForAllRequests_) needs to be enabled in web.config:

```
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true">
  </modules>
</system.webServer>
```

See the [samples](https://github.com/IdentityServer/IdentityServer3.Samples) repo for various examples of IIS and self-hosting.