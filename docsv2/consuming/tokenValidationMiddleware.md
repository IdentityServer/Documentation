---
layout: docs-default
---

# The Katana Access Token Validation Middleware

Consuming IdentityServer access tokens in web APIs is easy - you simply drop in our token validation middleware
into your Katana pipeline and set the URL to IdentityServer. All configuration and validation is done for you.

You can get the middleware here: [nuget](https://www.nuget.org/packages/IdentityServer3.AccessTokenValidation/)
or [source code](https://github.com/IdentityServer/IdentityServer3.AccessTokenValidation).

High level features:

* Support for validating both JWT and reference tokens
* Support for validating scopes
* Built-in configurable in-memory cache for caching reference token validation results
* Cache implementation can be replaced

The typical use case is, that you provide the URL to IdentityServer and the scope name of the API:

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        // turn off any default mapping on the JWT handler
        JwtSecurityTokenHandler.InboundClaimTypeMap = new Dictionary<string, string>();

        app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
            {
                Authority = "https://localhost:44333/core",
                RequiredScopes = new[] { "api1" }
            });

        app.UseWebApi(WebApiConfig.Register());
    }
}
```

The middleware will first inspect the token - if it is a JWT, token validation will be done locally on the receiving
server (using the issuer name and key material found in the discovery document).
If the token is a reference token, the middleware will use the access token validation [endpoint](../endpoint/accessTokenValidation.html) on IdentityServer.

## Enabling Caching

When using reference tokens, you probably don't want to make a round-trip to IdentityServer for each incoming request.
In this case you can cache the validation outcome locally.


```csharp
app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
    {
        Authority = "https://localhost:44333/core",
        RequiredScopes = new[] { "write" },

        EnableValidationResultCache = true,
        ValidationResultCacheDuration = TimeSpan.FromMinutes(10)
    });
```