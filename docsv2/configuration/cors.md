---
layout: docs-default
---

# CORS

Many endpoints in IdentityServer will be accessed via Ajax calls from JavaScript. Given that IdentityServer will most likely be hosted on a different origin than these clients, this implies that [Cross-Origin Resource Sharing](http://www.html5rocks.com/en/tutorials/cors/) (CORS) will be an issue.

## Cors Policy Service

IdentityServer3 allows the hosting application to implement a `ICorsPolicyService` to determine the CORS policy. This service is registered on the [`IdentityServerServiceFactory`](serviceFactory.html).

The single method on the `ICorsPolicyService` is:

* `Task<bool> IsOriginAllowedAsync(string origin)`
 * Returns `true` if the `origin` is allowed, `false` otherwise.

You can implement a custom implementation to determine in any way you see fit if the calling origin is allowed.

### Provided implementations

There are two implementations that are provided from IdentityServer core:

* `DefaultCorsPolicyService`
    * This implementation can be used if the list of allowed origins to allow is fixed and known at application start. The `AllowedOrigins` property is the collection that can be confgured with the list of  origins that should be allowed.
    * There is also a `AllowAll` property which can be set to `true` to allow all origins.
* `InMemoryCorsPolicyService`
    * This implementation accepts as a constructor argument a list of `Client` objects. The origins allowed for CORS  is configured via the `AllowedCorsOrigins` property of the `Client` objects. 
    * This implementaion will automatically be registered if the `UseInMemoryClients` configuration extension method is used. 

There is one last implementation provided from IdentityServer3.EntityFramework:

* `ClientConfigurationCorsPolicyService`
    * This implementation draws its list of allowed origins from the `AllowedCorsOrigins` property of the `Client` objects that are stored in the database.
    * This implementaion will automatically be registered if the `RegisterClientStore` or `RegisterConfigurationServices` extension methods are used. 

