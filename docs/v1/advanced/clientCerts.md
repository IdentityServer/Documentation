---
layout: docs-default
---

# Authenticating Clients using X.509 Certificates

**Remark** This functionality is currently only available on the dev branch and the v2 preview packages.

Often client authentication is accomplished using shared keys (aka client secrets). Another option is to use X.509
client certificates.

## Registering the client
You are in full control of how you want to map a client certificate to a client by implementing `IClientValidator`.
By default we will offer to use the thumbprint of the certificate to map to the right client.

The following snippet registers a client for client credentials flow:

```csharp
var certClient = new Client
{
    ClientName = "Client Credentials Flow Client with Client Certificate",                   
    ClientId = "certclient",
    
    ClientSecrets = new List<Secret>
    { 
        new Secret
        {
            Value = "61B754C541BBCFC6A45A9E9EC5E47D8702B78C29",
            Type = Constants.SecretTypes.X509CertificateThumbprint,
        }
    },

    Flow = Flows.ClientCredentials,
                    
    AllowedScopes = new List<string> 
    {
        "read", 
        "write"
    },
}
```

## Configuring the host

You need to configure your host to accept client certificates. For IIS you need to create a location element for the token
endpoint that configures the client cert and SSL settings:

```xml
<location path="core/connect/token">
  <system.webServer>
    <security>
      <access sslFlags="Ssl, SslNegotiateCert" />
    </security>
  </system.webServer>
</location>
```

**Remark** The SSL settings are locked down by default in IIS - you might need to set them to `Read/Write` in the feature
delegation configuration.

## Requesting the token

To request a token, you need to supply the client certificate to the HTTP client and add the client ID to the post body.
The following example uses the IdentityModel OAuth2 client:

```csharp
static TokenResponse RequestToken()
{
    var cert = new X509Certificate2("Client.pfx");

    var handler = new WebRequestHandler();
    handler.ClientCertificates.Add(cert);

    var client = new OAuth2Client(
        new Uri("https://idsrv.local/core/connect/token"),
        handler);


    var additional = new Dictionary<string, string>
    {
        { "client_id", "certclient" }
    };

    return client.RequestClientCredentialsAsync("read write", additional).Result;
}
```
