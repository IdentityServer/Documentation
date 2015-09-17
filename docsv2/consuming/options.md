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

## Static Configuration for JWTs
Instead of automatically obtaining the configuration from the discovery endpoint, you can also statically configure the middleware

* `IssuerName` set the expected issuer name of IdentityServer
* `SigningCertificate` sets the X.509 certificate to validate the access token signatur

## Enabling Caching

When using reference tokens, you probably don't want to make a round-trip to IdentityServer for each incoming request.
In this case you can cache the validation outcome locally.

* `EnableValidationResultCache` enables/disables caching of the validation outcome (defaults to `false`).
* `ValidationResultCacheDuration` set the cache duration (defaults to 5 minutes).
* `ValidationResultCache` sets the caching implemenation. Defaults to an in-memory cache but can be customized by
    implementing `IValidationResultCache`.