---
layout: docs-default
---

# Dependency Injection

**The sample for this topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/DependencyInjection)**

IdentityServer3 has extensibility points for various services.
The default implementations of these services are designed to be decoupled from other moving parts of IdentityServer
and as such we use dependency injection to get everything wired up.

## Injecting IdentityServer services

The default services provided by IdentityServer can be replaced by the hosting application if desired.
Custom implementations can also use dependency injection and have IdentityServer types or even custom types injected.
This is only supported with custom services and stores that are registered using a `Registration`.

For custom services to accept types defined by IdentityServer, simply indicate those dependencies as constructor arguments.
When IdentityServer instantiates your registered type the constructor arguments will be resolved automatically. For example:

```csharp
public class MyCustomTokenSigningService: ITokenSigningService
{
    private readonly IdentityServerOptions _options;

    public MyCustomTokenSigningService(IdentityServerOptions options)
    {
        _options = options;
    }

    public Task<string> SignTokenAsync(Token token)
    {
        // ...
    }
}
```

That was registered as such:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(typeof(MyCustomTokenSigningService));
```

or alternatively with this syntax:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
```

## Injecting custom services

Your custom services might also have dependencies on your own types.
Those can also be injected as long as they have been configured with IdentityServer’s dependency injection system.
This is done by adding new registrations using the `IdentityServerServiceFactory`’s `Register()` method.
For example, if you have a custom logger that is needed in your service:

```csharp
public interface ICustomLogger
{
    void Log(string message);
}

public class DebugLogger : ICustomLogger
{
    public void Log(string message)
    {
        Debug.WriteLine(message);
    }
}

public class MyCustomTokenSigningService: ITokenSigningService
{
    private readonly IdentityServerOptions _options;
    private readonly ICustomLogger _logger;

    public MyCustomTokenSigningService(IdentityServerOptions options, ICustomLogger logger)
    {
        _options = options;
        _logger = logger;
    }

    public Task<string> SignTokenAsync(Token token)
    {
        // ...
    }
}
```

Then it would be registered as such:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
factory.Register(new Registration<ICustomLogger, MyCustomDebugLogger>());
```

### Custom services without an interface

In the above example the injected type was the `ICustomLogger` and the implementation was the `MyCustomDebugLogger`. If your custom services is not designed with an interface to separate the contract from the implementation, then the concrete type itself can be registered to be injected.

For example, if the `MyCustomTokenSigningService`'s constructor did not accept an interface for the logger, as such:

```csharp
public class MyCustomTokenSigningService: ITokenSigningService
{
    public MyCustomTokenSigningService(IdentityServerOptions options, MyCustomDebugLogger logger)
    {
        _options = options;
        _logger = logger;
    }

    // ...
}
```

Then the registration could instead be configured like this:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService, MyCustomTokenSigningService>();
factory.Register(new Registration<MyCustomDebugLogger>());
```

In short, this type of registration means that the `MyCustomDebugLogger` concrete type is the dependency type to be injected.

## Customizing the creation

If it's necessary for your service to be constructed manually (e.g.  you need to pass specific arguments to the constructor), then you can use the `Registration` class that allows a factory callback to be used. This `Registration` has this signature:

```csharp
new Registration<T>(Func<IDependencyResolver, T> factory) 
```

The return value must be an instance of the `T` interface and an example might look like this:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService("SomeSigningKeyValue")
);
```

### Obtaining other dependencies

While this approach allows you the most flexibility to create your service, you still might require the use of other services. This is what the `IDependencyResolver` is used for. It allows you to get services from within your callback function. For example, if your custom client store requires a dependency on another service from within IdentityServer, it can be obtained as such:

```csharp
var factory = new IdentityServerServiceFactory();
factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService("SomeSigningKeyValue", resolver.Resolve<ICustomLogger>())
);
```

### Named dependencies

Finally, when registering custom dependencies via `IdentityServerServiceFactory`’s `Register()` method, the dependenciess can be named. This name is indicated via the `name` constructor parameter on the `Registration` class. This is only used for custom registrations and the name can only used when resolving dependencies with the `IDependencyResolver` from within the custom factory callback.

Named dependencies can be useful to register multiple instances of the same `T` yet provide a mechanism to distinguish the implementation desired. For example:

```csharp
string mode = "debug"; // or "release", for example

var factory = new IdentityServerServiceFactory();

factory.Register(new Registration<ICustomLogger, MyFileSystemLogger>("debug"));
factory.Register(new Registration<ICustomLogger, MyDatabaseLogger>("release"));

factory.TokenSigningService = new Registration<ITokenSigningService>(resolver =>
    new MyCustomTokenSigningService(resolver.Resolve<ICustomLogger>(mode))
);
```

In this example, the `mode` acts as a configuration flag to influence which impelmentation of the `ICustomLogger` will be used at runtime.

## Instancing with the Registration Mode

The `Registration` class allows a service to indicate how many instances of the service that will be created. The `Mode` property is an enum with these possible values:

* `InstancePerHttpRequest`
    One instance will be created per HTTP request. This means that if the service is requested twice in a single dependency chain then the same instance will be shared.
* `InstancePerUse`
    One instance will be created per location the service is needed. This means that if the service is requested twice in a single dependency chain then two separate instances will be created.
* `Singleton`
    Only one instance will ever be created. This mode is automatically assigned when using the `Registration` that accepts a singleton instance as the constructor parameter.
