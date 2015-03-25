---
layout: docs-default
---

# Packaging and Builds

IdentityServer consists of a number of nuget packages.

### Core

[nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer3)

Contains the core IdentityServer object model, services and server. Core only contains support for in-memory configuration and user stores - but you can plug-in support for other stores via the configuration. This is what the other repos and packages are about.

### Configuration stores
Storage of configuration data (clients and scopes) as well as runtime data (consent, token handles, refresh tokens).

Entity Framework [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3.EntityFramework/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer3.EntityFramework)

**(Community contribution)** MongoDb [github](https://github.com/jageall/IdentityServer.v3.MongoDb)

### User stores
Support for identity management libraries.

MembershipReboot [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3.MembershipReboot/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer3.MembershipReboot)

ASP.NET Identity [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3.AspNetIdentity/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer3.AspNetIdentity)

### Plugins
Protocol plugins.

WS-Federation [nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3.WsFederation/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer3.WsFederation)

### Access token validation middleware
OWIN middleware for APIs. Provides an easy way to validate access tokens and enforce scope requirements.

[nuget](https://www.nuget.org/packages/Thinktecture.IdentityServer3.AccessTokenValidation/) | [github](https://github.com/identityserver/Thinktecture.IdentityServer.v3.AccessTokenValidation)

## Dev builds

In addition we publish dev/interim builds to MyGet.
Add the following feed to your Visual Studio if you want to give them a try:

[https://www.myget.org/F/identity/](https://www.myget.org/F/identity/)
