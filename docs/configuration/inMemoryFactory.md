---
layout: docs-default
---

# The In-Memory Factory

The in-memory factory is an easy way to get a test/dev version of IdentityServer up and running.

When you create the in-mem factory you can supply a static list of `Client`, `Scope` and `InMemoryUser`.
All other data (e.g consent, refresh tokens, reference tokens, code etc.) will be kept in memory only.
This is only suitable for testing and development.

```csharp
var factory = InMemoryFactory.Create(
    users:   Users.Get(),
    clients: Clients.Get(),
    scopes:  Scopes.Get());

var idsrvOptions = new IdentityServerOptions
{
    Factory = factory,
    SigningCertificate = Cert.Load(),
};

app.UseIdentityServer(idsrvOptions);
```