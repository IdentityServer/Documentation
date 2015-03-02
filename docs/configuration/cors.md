---
layout: docs-default
---

#CORS

Many endpoints in IdentityServer will be accessed via Ajax calls from JavaScript. Given that IdentityServer will most likely be hosted on a different origin than these clients, this implies that [Cross-Origin Resource Sharing](http://www.html5rocks.com/en/tutorials/cors/) (CORS) will be an issue.

## Cors Policy Service

IdentityServer3 allows the hosting application to implement a `ICorsPolicyService` to determine the CORS policy. This service is registered on the [`IdentityServerServiceFactory`](serviceFactory.html).

The single method to implement is:
* `Task<bool> IsOriginAllowedAsync(string origin)`. 
 * Returns `true` if the `origin` is allowed, `false` otherwise.

There are two implementations that are provided from IdentityServer core:

* `DefaultCorsPolicyService`
  * This implementation can be used if the list of allowed origins to allow is fixed and known at application start. The `AllowedOrigins` propery is the collection that can be confgured with the list of origins that should be allowed. There is also a `AllowAll` property which can be set to `true` to allow all origins.

* `InMemoryCorsPolicyService`
  * This implementation accepts as a constructor argument a list of `Client` objects. The origins allowed for CORS  is configured via the `AllowedCorsOrigins` property of the `Client` objects. 

There is one last implementation provided from IdentityServer3.EntityFramework:
* `ClientConfigurationCorsPolicyService`
  * This implementation draws its list of allowed origins from the `AllowedCorsOrigins` property of the `Client` objects that are stored in the database.

## Deprecated: CorsPolicy

In version 1.0.0 of IdentityServer the `CorsPolicy` was the only means for supporting CORS and has now been deprecated in favor of the Cors Policy Service described above. The documentation below is maintained because the 1.0.0 feature is still supported, but will be removed in a future version.

### CorsPolicy

IdentityServer3 allows the hosting application to configure a `CorsPolicy` on the `IdentityServerOptions` to control which origins are allowed. 

#### AllowedOrigins

The `CorsPolicy` has two ways to indicate which origins are allowed. The first is the `AllowedOrigins` collection of host names. This is useful if at application start time the list of origins is known (either hard coded or perhaps loaded from a database).

```csharp
var idsvrOptions = new IdentityServerOptions();
idsrvOptions.CorsPolicy.AllowedOrigins.Add("http://myclient.com");
idsrvOptions.CorsPolicy.AllowedOrigins.Add("http://myotherclient.org
```

#### PolicyCallback

The second way the `CorsPolicy` allows the hosting application to indicate which origins are allowed is the `PolicyCallback` delegate which is a `Func<string, Task<bool>>`. This delegate is invoked at runtime when a CORS request is being made to IdentityServer and passed the origin being requested. A return `bool` value of `true` indicates that the origin is allowed. This is useful if the list of allowed origins changes frequently or is sufficiently large such that a database lookup is preferred.

```csharp
var idsvrOptions = new IdentityServerOptions();
idsrvOptions.CorsPolicy.PolicyCallback = async origin =>
{
    return await SomeDatabaseCallToCheckIfOriginIsAllowed(origin);
};
```

#### CorsPolicy.AllowAll

For convenience there is a static property `CorsPolicy.AllowAll` that will allow all origins. This is useful for debugging or development.

```csharp
var idsvrOptions = new IdentityServerOptions {
    CorsPolicy = CorsPolicy.AllowAll
};
```
