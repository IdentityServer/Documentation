---
layout: docs-default
---

# Schema Changes and Migrations

As IdentityServer3 is enhanced, it is likely that database schema changes will occur. In anticipation of schema changes, it is recommended (and expected) that the hosting application will be responsible for handling these schema changes over time.

Entity Framework provides migrations (more info [here](https://msdn.microsoft.com/en-us/data/jj591621.aspx) and [here](https://msdn.microsoft.com/en-us/data/dn481501)) as an approach to deal with schema changes and updating your database with those changes.

## DbContexts

There are three different `DbContext`-derived classes contained in the EF implementation. They are:

* `ClientConfigurationDbContext`
* `ScopeConfigurationDbContext`
* `OperationalDbContext`

These will be needed since the database context classes are in a different assembly (i.e. `IdentityServer3.EntityFramework`) than the hosting application.

## Enabling migrations

A migration must be created for each database context class. To enable migrations for all of the database context classes, the command below can be used from the the package manager console:

```
Enable-Migrations -MigrationsDirectory Migrations\ClientConfiguration -ContextTypeName ClientConfigurationDbContext -ContextAssemblyName IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config

Enable-Migrations -MigrationsDirectory Migrations\ScopeConfiguration -ContextTypeName ScopeConfigurationDbContext -ContextAssemblyName IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config

Enable-Migrations -MigrationsDirectory Migrations\OperationalConfiguration -ContextTypeName OperationalDbContext -ContextAssemblyName IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config
```

The the initial schema must then be defined (again one for each migration). Replace the tokens $ProjectRootNamespace$ by the actual root namespace of your project and create initial schema migrations like this:

```
Add-Migration -Name InitialCreate -ConfigurationTypeName $ProjectRootNamespace$.Migrations.ScopeConfiguration.Configuration -ConnectionStringName IdSvr3Config

Add-Migration -Name InitialCreate -ConfigurationTypeName $ProjectRootNamespace$.Migrations.ClientConfiguration.Configuration -ConnectionStringName IdSvr3Config

Add-Migration -Name InitialCreate -ConfigurationTypeName $ProjectRootNamespace$.Migrations.OperationalConfiguration.Configuration -ConnectionStringName IdSvr3Config
```

And then the database can be created, again replace $ProjectRootNamespace$ with your root namespace:

```
Update-Database -ConfigurationTypeName $ProjectRootNamespace$.Migrations.ClientConfiguration.Configuration -ConnectionStringName IdSvr3Config

Update-Database -ConfigurationTypeName $ProjectRootNamespace$.Migrations.ScopeConfiguration.Configuration -ConnectionStringName IdSvr3Config

Update-Database -ConfigurationTypeName $ProjectRootNamespace$.Migrations.OperationalConfiguration.Configuration -ConnectionStringName IdSvr3Config
```

Once your application updates to a new version, then you can use `Add-Migration` and `Update-Database` to update to the new schema. Check the EF [documentation](https://msdn.microsoft.com/en-us/data/jj591621.aspx) for more details.
