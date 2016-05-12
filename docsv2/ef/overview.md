---
layout: docs-default
---

# Entity Framework support for Clients, Scopes, and Operational Data

Support for Entity Framework for IdentityServer comes from [this](https://github.com/IdentityServer/IdentityServer3.EntityFramework) repo.

## Client and Scope configuration

If scope or client data is desired to be loaded from a database (rather than use in-memory configuration), then we provide a Entity Framework based implementations of the `IClientStore` and `IScopeStore` services. [More Information](clients_scopes.html).

## Operational data

Additionally, it is highly recommended that operational data (such as authorization codes, refresh tokens, reference tokens, and user consent) be persisted as well in a database. As such, we also have Entity Framework based implementations of the `IAuthorizationCodeStore`, `ITokenHandleStore`, `IRefreshTokenStore`, and `IConsentStore` services. [More Information](operational.html).

## Using Entity Framework migrations with SQL Azure

Entity Framework migrations can be used with SQL Azure, to do so, a custom `DbConfiguration` is required. The `DbConfiguration` is responsible for configuring the `DbExecutionStrategy`, which has to be set to `SqlAzureExecutionStrategy` [More Information](sql_azure.html).
