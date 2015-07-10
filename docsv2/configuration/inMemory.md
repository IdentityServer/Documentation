---
layout: docs-default
---

#In-Memory Services and Stores

The in-memory services and stores are an easy way to get a test/dev version of IdentityServer up and running.

If not specifically configured we will always provide an in-memory version stores for authorization codes, consent, 
reference and refresh tokens.

For clients, stores and users you need to supply a static list of `Client`, `Scope` and `InMemoryUser`.

**This is only suitable for testing and development.**

```csharp
var factory = new IdentityServerServiceFactory()
        .UseInMemoryUsers(Users.Get())
        .UseInMemoryClients(Clients.Get())
        .UseInMemoryScopes(Scopes.Get());

var idsrvOptions = new IdentityServerOptions
{
    Factory = factory,
    SigningCertificate = Cert.Load(),
};

app.UseIdentityServer(idsrvOptions);
```