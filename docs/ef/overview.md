---
layout: docs-default
---

#Entity Framework support for Clients, Scopes, and Operational Data

## Client and Scope configuration

If scope or client data is desired to be loaded from a database (rather than use in-memory configuration), then we provide a Entity Framework based implementations of the `IClientStore` and `IScopeStore` services. [More Information](clients_scopes.html).

## Operational data

Additionally, it is highly recommended that operational data (such as authorization codes, refresh tokens, reference tokens, and user consent) be persisted as well in a database. As such, we also have Entity Framework based implementations of the `IAuthorizationCodeStore`, `ITokenHandleStore`, `IRefreshTokenStore`, and `IConsentStore` services. [More Information](operational.html).
