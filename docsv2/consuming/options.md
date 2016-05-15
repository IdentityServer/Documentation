---
layout: docs-default
---

The `IdentityServerBearerTokenAuthenticationOptions` class has a number of options to influence token validation.

## General

* `Authority` sets the base address of IdentityServer. This allows for auto configuration (JWTs) and access to the
    token validation endpoint (reference tokens).
* `RequiredScopes` set the value of one **OR** more `scope` claims that are expected to be present in the access token. 
* `ValidationMode` can be either set to `Local` (JWTs only), `ValidationEndpoint` (JWTs and reference tokens using the validation
    endpoint - and `Both` for JWTs locally and reference tokens using the validation endpoint (defaults to `Both`).
* `TokenProvider` defines how to get the token from the HTTP request. By default the `Authorization` header is inspected.
    A custom token provider could be useful for getting the token from an alternative header or a query string.
* `NameClaimType` sets the name claim type of the `ClaimsIdentity` (defaults to `name`).
* `RoleClaimType` sets the role claim type of the `ClaimsIdentity` (defaults to `role`).
* `PreserveAccessToken` if set to true a claim of type `token` is created that holds the incoming access token (defaults to `false`).
   This is useful for situations where you want to forward the access token to another API, e.g. for calling the user info endpoint.
* `BackChannelHttpHandler` allows specifying a custom handler for all back-channel communication 
    (e.g. to the discovery endpoint or the token validation endpoint).
* `BackchannelCertificateValidator` allows specifying a custom certificate validator for the back-channel communication.
* `DelayLoadMetadata` tells the middleware to not load the metadata at application startup time, but on the first incoming request (defaults to `false`). This is useful when the discovery endpoint is not available at startup time - e.g. when the consumer is hosted in the same process as the token service.

## Static Configuration for JWTs
Instead of automatically obtaining the configuration from the discovery endpoint, you can also statically configure the middleware

* `IssuerName` set the expected issuer name of IdentityServer
* `SigningCertificate` sets the X.509 certificate to validate the access token signatur

**Remark** This is useful, if for some reaso the discovery document is not available to you, e.g. when running
IdentityServer and the client or API in the same web application.

## Using the Introspection Endpoint (added in v2.2)
Version 2.2 of IdentityServer added support for the token introspection specification. This is the recommended
technique when using reference tokens (see [here](../endpoints/introspection.html)).

In this case you need to specify the `ClientId` and `ClientSecret` to match the name and secret of the scope configuration
in IdentityServer (see [scopes](../configuration/scopesAndClaims.html))

## Enabling Caching

When using reference tokens, you probably don't want to make a round-trip to IdentityServer for each incoming request.
In this case you can cache the validation outcome locally.

* `EnableValidationResultCache` enables/disables caching of the validation outcome (defaults to `false`).
* `ValidationResultCacheDuration` set the cache duration (defaults to 5 minutes).
* `ValidationResultCache` sets the caching implemenation. Defaults to an in-memory cache but can be customized by
    implementing `IValidationResultCache`.
