---
layout: docs-default
---

#Federated post logout redirects

When a client application is signing out of IdentityServer, a "post-logout redirect uri" can be passed to request that the user is redirected back to the client application once they have fully signed out. This information is maintained in a "sign out message" cookie, which is identified by a unique "sign out message id". This "sign out message id" is passed as a query paramater to the "logged out" page so that the cookie can be accessed in order to provide the user a link to navigate back to the client application.

If IdentityServer is using upstream identity providers, then when a user is signed out of IdentityServer they will be redirected to the provider to signout. If the upstream identity provider supports a "post logout redirect uri" feature, then once the user is returned back to IdentityServer then the "sign in message id" parameter would be lost.

IdentityServer supports the ability for your code to obtain and store the "sign out message id" prior to redirecting to the upstream signout page. Then once the user has been redirected back to IdentityServer your code can use this "sign out message id" to display the IdentityServer "logged out" page, which will in turn use the "sign out message id" to display the link back to the original client application.

This requires your host to obtain the "sign out message id" prior to redirecting to the identity provider for signout. This is done by invoking `GetSignOutMessageId` from the IdentityServer [OWIN environment extensions](owin.html). Your code would then need to store this identifier (e.g. in a cookie).

Also, this requires your host to define a "post logout redirect" endpoint (which would be registered with the upstream identity provider) that acts as the callback once a user has logged out of the external identity provider. When it's invoked it can display IdentityServer's "logged out page" by invoking `RenderLoggedOutViewAsync` from the IdentityServer [OWIN environment extensions](owin.html). The "sign in message id" is a parameter to this method.

The code below shows an example:

```csharp
public void Configuration(this IAppBuilder app)
{
   app.Map("/core", coreApp =>
   {
      var factory = new IdentityServerServiceFactory();

      // ...

      coreApp.UseIdentityServer(idsrvOptions);
      
      coreApp.Map("/signoutcallback", cleanup =>
      {
         cleanup.Run(async ctx =>
         {
            var state = ctx.Request.Cookies["state"];
            await ctx.Environment.RenderLoggedOutViewAsync(state);
         });
      });
   });
}


 public static void ConfigureIdentityProviders(IAppBuilder app, string signInAsType)
{
    var oidc = new OpenIdConnectAuthenticationOptions
    {
        AuthenticationType = "oidc",
        Caption = "External IdP",
        SignInAsAuthenticationType = signInAsType,
        // ...
        Notifications = new OpenIdConnectAuthenticationNotifications
        {
            RedirectToIdentityProvider = n =>
            {
                if (n.ProtocolMessage.RequestType == Microsoft.IdentityModel.Protocols.OpenIdConnectRequestType.LogoutRequest)
                {
                    // ...
                
                    var signOutMessageId = n.OwinContext.Environment.GetSignOutMessageId();
                    if (signOutMessageId != null)
                    {
                        n.OwinContext.Response.Cookies.Append("state", signOutMessageId);
                    }
                }
                return Task.FromResult(0);
            }
        }
    };
    app.UseOpenIdConnectAuthentication(oidc);
}
```
