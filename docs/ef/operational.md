---
layout: docs-default
---

#Operational Data

For many of IdentityServer3's features a database is required to persist various operational data. This includes authorization codes, refresh tokens, reference tokens, and user consent. 

## Registration

There are a variety of stores that will persist the operational data. These are configured with a single extension method on `IdentityServerServiceFactory`. All of the extension methods accept an `EntityFrameworkServiceOptions` which contains these properties:

* `ConnectionString`: The name of the connection string as configured in the `.config` file.
* `Schema`: An optional database schema to use for the tables. If not provided, then the database default will be used.

To configure the operational data stores this code could be used:

```csharp
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterOperationalServices(efConfig);
``` 

## Data Cleanup

Much of the operational data has an expiration. It is likely desirable to remove this stale data after. This can be done outside of the application that hosts IdentityServer or by the database itself (via various mechanisms). If it is desired that application code that you perform this cleanup, then the `TokenCleanup` class is provided to assist. It accepts a `EntityFrameworkServiceOptions` and an `Int32` interval (in seconds) to configure how frequently the stale data is cleared. It will asynchronously connect to the database and can be configured as such:

```csharp
var efConfig = new EntityFrameworkServiceOptions {
    ConnectionString = connString,
    //Schema = "foo"
};

var cleanup = new TokenCleanup(efConfig, 10);
cleanup.Start();
``` 
