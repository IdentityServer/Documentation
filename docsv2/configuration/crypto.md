---
layout: docs-default
---

# Keys, Signatures and Cryptography
IdentityServer depends on cryptography in various places. Here's an overview.

## Secure Transport (HTTPS)
By default, IdentityServer requires all incoming connections to come over HTTPS. It is absolutely mandatory that
communication with IdentityServer is done only over secured transports. There are certain deployment scenarios like
SSL offloading where this requirement can be relaxed. See the [deployment](../advanced/deployment.html) section for more information.

SSL/TLS configuration is done at the hosting level - e.g in IIS or with HTTP.SYS directly.

## Token Signing
Identity and access tokens are signed with an X.509 certificate using the RSA algorithm (RS256).
The signing certificate is set on the `IdentityServerOptions` using the `SigningCertificate` property.
Setting this property is mandatory for identity tokens and JWT access tokens. 
If you are using reference tokens only, you don't need to set the signing certificate.

### Signing Key Rollover
X.509 certificates have a finite lifetime. Renewing and rolling over the singing key without downtime of your application 
can be a challenge.
IdentityServer has a couple of features that make this process easier:

* The discovery document publishes the current (and a secondary) public key. This way token consumers can learn about the key material
* All JWTs contain a key identifier that matches the key pubished in the discovery document
* The access token validation middleware periodically (every 24h) checks the discovery document to update its own key configuration

A key rollover could work like this:

1. Get the new certificate that should replace the old one
2. Set this new certificate as the `SecondarySigningCertificate` on the options. IdentityServer will now publish both 
certificates in the discovery document and the access token validation middleware will accept tokens signed with both keys
3. Wait at least 24h to give every consumer a chance to update its configuration
4. Set the new certificate as the primary `SigningCertificate`. Retire the old certificate

## Cookie Protection
Cookies must be protected as well. IdentityServer uses the Katana data protection infrastructure for that.
If you application is hosted in IIS, Katana wil use the ASP.NET machine key to protect all cookies. If deployed in a 
web farm you need to manually synchronize those keys over all nodes.

If self hosted, you can either plug in any custom Katana data protection provider in your host, or use the built-in 
IdentityServer data protector based on X.509 certificates.