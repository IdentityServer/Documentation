---
layout: docs-default
---

#OWIN Environment Extensions

IdentityServer3 provides extension methods for the OWIN environment (`IDictionary<string, object>`) in the `IdentityServer3.Core.Extensions` namespace). These can be used to access features of IdentityServer from custom pages running in the same host as IdentityServer. Note that these can only be used when the custom pages are running in the same OWIN pipeline as IdentityServer.

* `GetIdentityServerHost`
* `GetIdentityServerBasePath`
* `GetIdentityServerBaseUrl`
* `GetCurrentUserDisplayName`
* `CreateSignInRequest`
* `IssueLoginCookie`
* `CreateSignOutRequest`
* `GetIdentityServerFullLoginAsync`
* `GetIdentityServerPartialLoginAsync`
* `GetPartialLoginRestartUrlAsync`
* `GetPartialLoginResumeUrlAsync`
* `UpdatePartialLoginClaimsAsync`
* `RemovePartialLoginCookie`
* `GetIdentityServerLogoutUrl`
* `GetSignInMessage`
* `GetRequestId`