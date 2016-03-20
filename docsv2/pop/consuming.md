---
layout: docs-default
---

# Consuming PoP tokens in a Katana pipeline

Consuming PoP tokens in a web api is a two-part configuration.
The first part is configuring your token validation middleware normally, except with some slight modifications to accommodate how PoP tokens are passed.
The second part is configuring middleware to validate the current HTTP request's signature against the proof of possession key.

The _IdentityModel.Owin.PopAuthentication_ NuGet package provides the necessary code to accomplish this configuration.

## Changes to the normal token validation middleware

The two main changes that need to be done to accommodate PoP tokens is to change the middleware's authentication scheme to `"PoP"` and
to configure a token provider to locate the access token within the PoP token.

To configure the authentication middleware to use the `"PoP"` scheme is simple -- it just involves setting the `AuthenticationType` property to `"PoP"` (notice both instances of the letter `P` must be uppercase).

To configure a token provider to locate the access token within the PoP token, you will need to set the `Provider` property and handle the `OnRequestToken` event.
There is a helper method `DefaultPopTokenProvider.GetAccessTokenFromPopTokenAsync` that can perform this work for you.

If you are using the _IdentityServer3.AccessTokenValidation_ middleware, then the configuration changes will look like this:

```csharp
public void Configuration(IAppBuilder app)
{
    JwtSecurityTokenHandler.InboundClaimTypeMap.Clear();

    app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
    {
        // we're looking for the PoP scheme, not Bearer
        AuthenticationType = "PoP",
        
        // locate the access token from within  the PoP token
        Provider = new OAuthBearerAuthenticationProvider
        {
            OnRequestToken = async ctx =>
            {
                ctx.Token = await DefaultPopTokenProvider.GetAccessTokenFromPopTokenAsync(ctx.OwinContext.Environment);
            }
        },
        
        // ...
    };
}
``` 

If you are using the _Microsoft.Owin.Security.Jwt_ middleware, then the configuration will look like this:

```csharp
public void Configuration(IAppBuilder app)
{
    JwtSecurityTokenHandler.InboundClaimTypeMap.Clear();

    app.UseJwtBearerAuthentication(new JwtBearerAuthenticationOptions
    {
        // we're looking for the PoP scheme, not Bearer
        AuthenticationType = "PoP",
        
        // locate the access token from within  the PoP token
        Provider = new OAuthBearerAuthenticationProvider
        {
            OnRequestToken = async ctx =>
            {
                ctx.Token = await DefaultPopTokenProvider.GetAccessTokenFromPopTokenAsync(ctx.OwinContext.Environment);
            }
        },  
              
        // ...
    };
}
```
 
## Middleware to validate HTTP request against proof of possession key

Once the above code is able to locate and validate the access token from within the PoP token, the HTTP request must be validated.
To perform this validation, simply register the `HttpSignatureValidationMiddleware` in the Katana pipeline after the access token validation middleware.

It will look like this:

```csharp
public void Configuration(IAppBuilder app)
{
    JwtSecurityTokenHandler.InboundClaimTypeMap.Clear();

    app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
    {
        // we're looking for the PoP scheme, not Bearer
        AuthenticationType = "PoP",
        
        // locate the access token from within  the PoP token
        Provider = new OAuthBearerAuthenticationProvider
        {
            OnRequestToken = async ctx =>
            {
                ctx.Token = await DefaultPopTokenProvider.GetAccessTokenFromPopTokenAsync(ctx.OwinContext.Environment);
            }
        },
        
        // ...
    };
    
    app.UseHttpSignatureValidation();
}
``` 

The default validation will validate the incoming access token and the timestamp contained in the PoP token. 
If you require signature validation on other aspects of the HTTP request, then a `HttpSignatureValidationOptions` can be passed to the `UseHttpSignatureValidation` API.

 The `HttpSignatureValidationOptions` contains these properties for configuring the request validation:
 * `TimespanValidityWindow`: `TimeSpan` for which the PoP token's `Timespan` will be valid;
 * `ValidateMethod`: Validate the HTTP request's method.
 * `ValidateHost`: Validate the HTTP request's host.
 * `ValidatePath`: Validate the HTTP request's path.
 * `ValidateBody`: Validate the HTTP request's body.
 * `QueryParametersToValidate`: Collection that indicates the query string keys to validate on HTTP request.
 * `RequestHeadersToValidate`: Collection that indicates the header keys to validate on HTTP request.

A sample can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/dev/source/Clients/SampleAspNetWebApiWithPop).
