---
layout: docs-default
---

# Entity Framework Support for Relying Parties

Support for Entity Framework for the IdentityServer WS-Federation plugin can be found in the [IdentityServer3.WsFederation.EntityFramework repository](https://github.com/IdentityServer/IdentityServer3.WsFederation.EntityFramework).

## Relying Party Configuration

The `RelyingPartyService` is the EF-based implementation of the `IRelyingPartyService` interface.

### Registration

To register the Relying Party store within IdentityServer the `RegisterRelyingPartyService` extension method on the `WsFederationServiceFactory` can be used. This extension method accepts an `EntityFrameworkServiceOptions` which contains these properties:

* `ConnectionString`: The name of the connection string as configured in the `.config` file.
* `Schema`: An optional database schema to use for the tables. If not provided, then the database default will be used.

Example configuration:

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new WsFederationServiceFactory(options.Factory);
factory.RegisterRelyingPartyService(efConfig);
```
