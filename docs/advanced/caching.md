---
layout: docs-default
---

#Caching results for client, scope, and user stores

The various stores allow IdentityServer to load data from a database. Given the general decoupled design of IdentityServer, the APIs to load data might be invoked several times in the same HTTP request into IdentityServer. This might incur unnecessary round trips to a database. Given this possibility, IdentityServer defines a caching interface so that you can implement your own caching logic. Additionally, IdentityServer provides a default cache implementation. 

## Default cache

The default cache provided by IdentityServer is an in-memory cache and is configurable with a duration for cached data to expire. This default cache (while fairly simple) should address most performance concerns. To configure this default cache, there are extension methods on the `IdentityServerServiceFactory`:

* `ConfigureClientStoreCache`
* `ConfigureScopeStoreCache`
* `ConfigureUserServiceCache`

These are overloaded to either accept no parameters (which use a default expiration of 5 minutes), or a `TimeSpan` that indicates a custom value for the expiration. These APIs would be used something like this:

```
var factory = InMemoryFactory.Create(
    users:   Users.Get(),
    clients: Clients.Get(),
    scopes:  Scopes.Get());

factory.ConfigureClientStoreCache();
factory.ConfigureScopeStoreCache();
factory.ConfigureUserServiceCache(TimeSpan.FromMinutes(1));
```

These methods should be called after you have assigned the appropriate store into the `IdentityServerServiceFactory`, as they use a decorator pattern to wrap the actual store implementation.

## Custom cache

If a custom cache impelmentation is desired (e.g. using reddis), then you can implement the `ICache<T>` for the data that needs to be cached. The cache interface defines this API:

* `Task SetAsync(string key, T item)`
 * The `key` and the `item` to cache.
* `Task<T> GetAsync(string key)`
 * The `key` indicates the item to access from the cache. A `null` returned item indicates there is no entry in the cache.

The `IdentityServerServiceFactory` extension methods described above are also overloaded to also accept a `Registration` for the custom cache:

* `ConfigureClientStoreCache(Registration<ICache<Client>> cacheRegistration)`
* `ConfigureScopeStoreCache(Registration<ICache<IEnumerable<Scope>>> cacheRegistration)`
* `ConfigureUserServiceCache(Registration<ICache<IEnumerable<Claim>>> cacheRegistration)`

