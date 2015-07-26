---
layout: docs-default
---

# Secrets

Secrets define how machines (e.g. a client or a scope) can authenticate with IdentityServer.

Example of a secret definition for a client:

```csharp
var client = new Client
{
    ClientName = "Client Credentials Flow Client",
    
    ClientId = "client",
    ClientSecrets = new List<ClientSecret>
    { 
        new ClientSecret("secret".Sha256())
    },

    Flow = Flows.ClientCredentials   
}
```

The above snippets sets a shared secret of value `secret` - and hashes it with SHA256.
The `ClientSecret` property is a list, which indicates that a client can have more than one secret.
This is useful for key rotation.

Let's have a look at the `Secret` class in more detail:

* `Value` The value of the secret. This is being interpreted by the secret validator (e.g. a "password"-like share secret
    or something else that identifies a credential)
* `Description` The description of the secret - useful for attaching some extra information to the secret
* `Expiration` A point in time, where this secret will expire
* `Type` Some string that gives the secret validator a hint what type of secret to expect (e.g. "SharedSecret" or "X509CertificateThumbprint")

## Secret parsers
There are a number of ways how a client could transmit a secret - the OAuth2 specification mentions HTTP Basic Authentication
or using POST body values. IdentityServer additionally supports X.509 client certificates (see [here](../advanced/clientCerts.html)).

Secret parsing is an extensibility point - if you need to support different means of transmitting a secret 
besides the above mentioned mechanisms, you can implement the `ISecretParser` interface and add your
implementation to the `SecretParsers` collection on the service factory.

## Secret validation
After the secret has been extracted from the incoming request, it must be validated.
By default we support shared secrets stored either using SHA256 or SHA512 hashing and X.509 certificates (by default
the certificate thumbprint is used to authenticate the caller).

Secret validation is also an extensibility point - if you need to support different validation methods, e.g. X.509 certificates
distinguished names with chain/peer trust, you can implement the `ISecretValidator` interface and add your
implementation to the `SecretValidators` collection on the service factory.