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

```
var efConfig = new EntityFrameworkServiceOptions {
   ConnectionString = "SomeConnectionName",
   //Schema = "someSchemaIfDesired"
};

var factory = new IdentityServerServiceFactory();
factory.RegisterOperationalServices(efConfig);
``` 
