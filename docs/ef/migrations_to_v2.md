---
layout: docs-default
---

#Schema Changes and Migrations

As IdentityServer3 is enhanced, it is likely that database schema changes will occur. In anticipation of schema changes, it is recommended (and expected) that the hosting application will be responsible for handling these schema changes over time.

Entity Framework provides [migrations](https://msdn.microsoft.com/en-us/data/jj591621.aspx) as an approach to deal with schema changes and updating your database with those changes.

## DbContexts

There are three different `DbContext`-derived classes contained in the EF implementation. They are:

* `ClientConfigurationDbContext`
* `ScopeConfigurationDbContext`
* `OperationalDbContext`

These will be needed since the database context classes are in a different assembly (i.e. `Thinktecture.IdentityServer3.EntityFramework`) than the hosting application.

## Enabling migrations

A migration must be created for each database context class. To enable migrations for all of the database context classes, the command below can be used from the the package manager console:

```
Enable-Migrations -MigrationsDirectory Migrations\ClientConfiguration -ContextTypeName ClientConfigurationDbContext -ContextAssemblyName Thinktecture.IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config

Enable-Migrations -MigrationsDirectory Migrations\ScopeConfiguration -ContextTypeName ScopeConfigurationDbContext -ContextAssemblyName Thinktecture.IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config

Enable-Migrations -MigrationsDirectory Migrations\OperationalConfiguration -ContextTypeName OperationalDbContext -ContextAssemblyName Thinktecture.IdentityServer3.EntityFramework -ConnectionStringName IdSvr3Config
```

The the initial schema must then be defined (again one for each migration), as such:

```
Add-Migration -Name InitialCreate -ConfigurationTypeName Host.Migrations.ScopeConfiguration.Configuration -ConnectionStringName IdSvr3Config

Add-Migration -Name InitialCreate -ConfigurationTypeName Host.Migrations.ClientConfiguration.Configuration -ConnectionStringName IdSvr3Config

Add-Migration -Name InitialCreate -ConfigurationTypeName Host.Migrations.OperationalConfiguration.Configuration -ConnectionStringName IdSvr3Config
```

And then the database can be created:

```
Update-Database -ConfigurationTypeName Host.Migrations.ClientConfiguration.Configuration -ConnectionStringName IdSvr3Config

Update-Database -ConfigurationTypeName Host.Migrations.ScopeConfiguration.Configuration -ConnectionStringName IdSvr3Config

Update-Database -ConfigurationTypeName Host.Migrations.OperationalConfiguration.Configuration -ConnectionStringName IdSvr3Config
```

Once your application updates to a new version, then you can use `Add-Migration` and `Update-Database` to update to the new schema. Check the EF [documentation](https://msdn.microsoft.com/en-us/data/jj591621.aspx) for more details.

## Migrating from 1.x to 2.0

IdentityServer 2.0.0 changed some client semantics, so there is a necessary EF migration. Unfortunately the EF migration tool does not understand all of these semantics, but fortunately this is not that difficult with some manual tweaks. Here is a link to what the final migration should look like:

[https://github.com/IdentityServer/IdentityServer3.Samples/blob/dev/source/EntityFramework/SelfHost/Migrations/ClientConfiguration/201504070205330_v2_0_0.cs](https://github.com/IdentityServer/IdentityServer3.Samples/blob/dev/source/EntityFramework/SelfHost/Migrations/ClientConfiguration/201504070205330_v2_0_0.cs)
