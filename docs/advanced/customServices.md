---
layout: docs-default
---

#Custom Services

IdentityServer3 provides many extensibility points for storage of data, validation logic and general functionality
that are needed to support IdentityServer's operation as a token service.
These various extensibility points are collectively referred to as "services".

See [here](../configuration/serviceFactory.html) for the full list of the services.

## Mandatory services
Three of the services are mandatory and must be configured by the implementer, they are:

* the user service (`IUserService`)

* the client store (`IClientStore`)

* the scope store (`IScopeStore`)

We provide a simple in-memory version of these three services as well as support for data stores via the related repos or community projects.
See [here](../configuration/inMemoryFactory.html) for more details.

##Registering custom Services

You can replace every service and register additional custom ones. This is encapsulated by the `Registration` class.
A `Registration` represents a way for IdentityServer to obtain an instance of your service.

Depending upon the design of your service you might want to have a new instance on every request, use a singleton,
or you might require special instantiation logic each time an instance is needed.
To accommodate these different possibilities, the `Registration` class provides many different constructors to register your service:

* `new Registration<T>(Type yourImplementation)`
    * Registers `yourImplementation` as the class that implements the `T` interface.
* `new Registration<T, Impl>()`
    * Registers `Impl` as the class that implements the `T` interface. This API is simply a convenience for the previous that accepts a `Type` parameter.
* `new Registration<T>(T singleton)`
    * Registers the `singleton` instance passed as a singleton implementation of the `T` interface.
* `new Registration<T>(Func<IDependencyResolver, T> factory)`
    * Registers a callback function that will be invoked to return an implementation of the `T` interface.

```csharp
var factory = new IdentityServerServiceFactory();
factory.UserService = new Registration<IUserService, MyCustomUserService>();
```

See the [DI](di.html) page for more details.

### Service cleanup

In all cases except for the singleton, if your service implements `IDisposable`, then `Dispose` will be called at the end of the HTTP request.


