---
layout: docs-default
---

# Deployment

**note** This document is work in progress. Feel free to add new content.

## Data Protection

If you are running in IIS, you need to synchronize machine keys. If you are running outside of IIS,
you need to use a web farm compatible data protector for Katana.

Unfortunately, Katana does not ship with one out of the box. IdentityServer includes a data protector based on
X.509 certificates (`X509CertificateDataProtector`) that you can set on the options class.

## Terminating SSL

If you want to terminate SSL on the load balancer, there are two relevant settings on the options:

 * `RequireSsl`
     * Set this to false to allow non-SSL connections between the load balancer and IdentityServer.
 * `PublicOrigin`
    * Since your internal farm nodes have different names than the public reachable address, IdentityServer can't use it
      for link generation. Sett this property to the public name.

## Signing Keys

Make sure the signing certificates are deployed to all nodes.

## Configuration data

The configuration data for scopes, clients and users must be in-sync.

Either they are static and you change configuration data via continuous deployment, or you use a persistence layer like
the Entity Framework repo (or community contributions like the one for MongoDB).

## Operation data

Some features require a shared database for operational data - namely authorization codes, reference tokens and refresh token.
If you use any of those features you need a persistence layer for them. Again you could use our Entity Framework implemenation.

## Caching

IdentityServer has a simple built-in in-memory cache. This is useful on its own but not as optimized as it could be for web farms.
You can plug in your own cache by implementing the `ICache` interface. Check the community contributions (e.g. for a Redis implementation).

