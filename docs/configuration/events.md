---
layout: docs-default
---

#Events

IdentityServer raises a number of events at runtime, e.g:

* Successful/failed authentication (resource owner flow, pre, partial, local and external)
* Token issued (identity, access, refresh tokens)
* Token handle related events (authorization code, refresh token issued/redeemed/refreshed)
* Permission revoked
* Endpoint success/failures
* Expired/invalid/no signing certificate
* Unhandled exceptions and internal errors

By default these events are forwarded to the configured log provider -
a custom event service can process or forward them in any way suitable for the environment.

##Configuring events
The `EventsOptions` class has the following settings (all default to `false`):

* `RaiseSuccessEvents`
    * e.g. refresh token refreshed or authentication success
* `RaiseFailureEvents`
    * e.g. authentication failure, authorization code redeem failure
* `RaiseErrorEvents`
    * e.g. unhandled exceptions
* `RaiseInformationEvents`
    * e.g. token issued or certificate valid
    * e.g. token issued or certificate valid
    