---
layout: docs-default
---

# Packaging and Builds

IdentityServer consists of a number of nuget packages.

### Core
Contains the core IdentityServer object model, services and server. Core is packaged as an OWIN module and can be hosted like this:

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseIdentityServer(options);
    }
}
```

Core only contains support for in-memory configuration and user stores - but you can plug-in support for other store via the `options`.

This is what the other repos and packages are about.

[nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3)

### Configuration stores

Entity Framework [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3.EntityFramework/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.EntityFramework)

### User stores

MembershipReboot [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3.MembershipReboot/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.MembershipReboot)

ASP.NET Identity [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3.AspNetIdentity/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.AspNetIdentity)

### Plugins

WS-Federation [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3.WsFederation/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.WsFederation)

### Access token validation middleware

[nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer.v3.AccessTokenValidation/) | [github](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.AccessTokenValidation)

## Dev builds

In addition we publish dev/interim builds to MyGet.
Add the following feed to your Visual Studio if you want to give them a try:

[https://www.myget.org/F/thinktecture/](https://www.myget.org/F/thinktecture/)
