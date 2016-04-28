---
layout: docs-default
---

# Signout support for server-side web applications

The [HTTP-based logout](https://openid.net/specs/openid-connect-logout-1_0.html) specification defines a mechanism for an OpenID Connect provider to inform client applications that a user has signed out. This is performed by creating an `<iframe>` to a well-known "logout URI" in each client application that the user has signed into. These `<iframe>`s are rendered on the "signed out" page at the OpenID Connect provider. This allows the `<iframe>
    `s to be executed in the context of the user's browser, thus allowing each client application to clear the user's session (whatever that means for the application).

To enable this for a client application, the `Client` configuration must have set the `LogoutUri` configuration property. By default, the user's session id is passed as a `sid` query string parameter and it intended to allow the client application to authenticate the signout notification. This `sid` parameter can be disabled by setting `LogoutSessionRequired` to `false` on the `Client` configuration.
    
A MVC client application would then need to handle the signout request with something like this:

```csharp
public void SignoutCleanup(string sid)
{
    var cp = (ClaimsPrincipal)User;
    var sidClaim = cp.FindFirst("sid");
    if (sidClaim != null && sidClaim.Value == sid)
    {
        Request.GetOwinContext().Authentication.SignOut("Cookies");
    }
}

```

**To use this technique for signout notification, consult the sample MVC application [here](https://github.com/IdentityServer/IdentityServer3.Samples/blob/master/source/Clients/MVC%20OWIN%20Client/Controllers/HomeController.cs#L50).**
