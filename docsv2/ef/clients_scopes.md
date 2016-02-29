---
layout: docs-default
---

# Clients and Scopes

## Stores

### ClientStore

The `ClientStore` is the EF-based implementation of the `IClientStore` interface. It can be used independently from the `ScopeStore`.  

### ScopeStore

The `ScopeStore` is the EF-based implementation of the `IScopeStore` interface. It can be used independently from the `ClientStore`. 

## Registration

To use either of the stores, they need to be registered. There are extension methods on the `IdentityServerServiceFactory` that allow either or both of the stores to be configured. All of the extension methods accept an `EntityFrameworkServiceOptions` which contains these properties:

* `ConnectionString`: The name of the connection string as configured in the `.config` file.
* `Schema`: An optional database schema to use for the tables. If not provided, then the database default will be used.

To configure the stores independently, this code could be used:

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterClientStore(efConfig);
factory.RegisterScopeStore(efConfig);
``` 

If both stores will be used with the same `EntityFrameworkServiceOptions`, then a single convenience extension method is provided:

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterConfigurationServices(efConfig);
``` 
