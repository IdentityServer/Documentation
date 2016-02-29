---
layout: docs-default
---

# Logout Endpoint

Redirecting to the logout endpoint clears the authentication session and cookie.

You can pass the following optional parameters to the endpoint:

* `id_token_hint`
    * The id_token that the client acquired during authentication.
     This allows bypassing the logout confirmation screen as well as providing a post logout redirect URL
* `post_logout_redirect_uri`
    * A URI that IdentityServer can redirect to after logout (by default a link is displayed). The URI must be in the list of allowed post logout URIs for the client.


```
/connect/endsession?id_token_hint=...&post_logout_redirect_uri=https://myapp.com
```

See the [AuthenticationOptions](../configuration/authenticationOptions.html) for configuring the behavior of the logout endpoint and logout page.
