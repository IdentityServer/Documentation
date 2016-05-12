---
layout: docs-default
---

# Using Entity Framework migrations with SQL Azure

In order to connect IdentityServer3.EntityFramework with SQL Azure, `SqlAzureExecutionStrategy` has to be configured. To use an alternative `DbExecutionStrategy`, a custom `DbConfiguration` class is required. Create a new class file and set the execution strategy as shown in the snippet below

```csharp

    using System.Data.Entity;
    using System.Data.Entity.SqlServer;

    namespace IdentityServer3.Samples.EntityFramework
    {

        public class MyDbConfiguration: DbConfiguration
        {
            SetExecutionStrategy("System.Data.SqlClient", () => new SqlAzureExecutionStrategy());
        }
    }

```

Having the custom `MyDbConfiguration`, you need to create derived classes for the `DbContext` classes you're interested in. By using the `DbConfigurationType` attribute both required pieces of code will be connected.

## Custom ClientConfigurationDbContext

```csharp

    using System.Data.Entity;
    using IdentityServer3.EntityFramework;


    namespace IdentityServer3.Samples.EntityFramework
    {
        [DbConfigurationType(typeof(MyDbConfiguration))]
        public class MyClientConfigurationDbContext : ClientConfigurationDbContext
        {
            public MyClientConfigurationDbContext()
            {

            }
            public MyClientConfigurationDbContext(EntityFrameworkServiceOptions entityFrameworkConfig):
                base(connectionString, schema)
            {

            }

            public MyClientConfigurationDbContext(string connectionString, string schema):
                base(connectionString, schema)
            {

            }
        }
    }

```

## Custom ScopeConfigurationDbContext

```csharp

    using System.Data.Entity;
    using IdentityServer3.EntityFramework;


    namespace IdentityServer3.Samples.EntityFramework
    {
        [DbConfigurationType(typeof(MyDbConfiguration))]
        public class MyScopeConfigurationDbContext : ScopeConfigurationDbContext
        {
            public MyScopeConfigurationDbContext()
            {

            }
            public MyScopeConfigurationDbContext(EntityFrameworkServiceOptions entityFrameworkConfig):
                base(entityFrameworkConfig.ConnectionString, entityFrameworkConfig.Schema)
            {

            }

            public MyScopeConfigurationDbContext(string connectionString, string schema):
                base(connectionString, schema)
            {

            }
        }
    }

```

## Custom OperationalDbContext

```csharp

    using System.Data.Entity;
    using IdentityServer3.EntityFramework;


    namespace IdentityServer3.Samples.EntityFramework
    {
        [DbConfigurationType(typeof(MyDbConfiguration))]
        public class MyOperationalDbContext : OperationalDbContext
        {
            public MyOperationalDbContext()
            {

            }
            public MyOperationalDbContext(EntityFrameworkServiceOptions entityFrameworkConfig):
                base(entityFrameworkConfig.ConnectionString, entityFrameworkConfig.Schema)
            {

            }

            public MyOperationalDbContext(string connectionString, string schema):
                base(connectionString, schema)
            {

            }
        }
    }

```
