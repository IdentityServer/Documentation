---
layout: docs-default
---

# Requesting PoP Tokens

**Remark** this is beta documentation.

PoP access tokens can be requested as part of an authorization code or refresh token flow.

The specifications allow a number of variations, but IdentityServer right now only supports client generated
asymmetric proof keys.

In a nutshell this works like this:

1. client generates an asymmetric key pair (e.g. RSA or EC)
2. client sends the public key part to identityserver along with the token request
3. identityserver will embed the key into the access token (inside the *cnf* claim)
4. client will use the private key to sign the HTTP requests, the resource server will use the public key from the
access token to verify the signatures

## Generating a key pair

```csharp
// create key pair
var p = RsaPublicKeyJwk.CreateProvider();
var key = p.ExportParameters(false);

// package public key part as a JWK
var jwk = RsaPublicKeyJwk.CreateJwk(key);
var jwk64 = RsaPublicKeyJwk.CreateJwkString(jwk);
```

## Requesting a PoP token

First request a token via authorization code flow - then when exchanging the token, send the public key along.

```csharp
var tokenClient = new TokenClient(
  IdentityServerPipeline.TokenEndpoint, 
  ClientId, 
  ClientSecret);

var tokenResponse = await tokenClient.RequestAuthorizationCodePopAsync(
  authResponse.Code,
  ClientRedirectUri,
  key: jwk64,
  algorithm: jwk.alg);
```

## Sending the token

Our PoP client library takes care of signing the outgoing HTTP requests

```csharp
// configure signing details
var signature = new RS256Signature(p);
var signingOptions = new RequestSigningOptions();
var signingHandler = new HttpSigningMessageHandler(signature, signingOptions);

// set pop token
var client = new HttpClient(signingHandler);
client.SetToken("PoP", tokenResponse.AccessToken);

// call API
var apiResponse = await client.GetAsync(WebApiPipeline.Endpoint);
```

The full source code can be found [here](https://github.com/IdentityModel/IdentityModel.Owin.PopAuthentication/blob/dev/src/IdentityModel.Owin.PopAuthentication.Tests/IntegrationTests/IdSvr_Client_And_WebApi_Integration/PopTests.cs).
Also a sample can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/dev/source/Clients/WpfOidcClientPop).
