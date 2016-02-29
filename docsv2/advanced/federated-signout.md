---
layout: docs-default
---

# Federated Signout

IdentityServer supports the ability to federate with external identity providers. When a user signs out of an upstream identity provider, depending upon the protocol used, it might be possible to receive a notification when the user has signed out. This would then allow IdentityServer to notify its clients so they can also sign the user out.

A common approach in both WS-Federation and OIDC providers is to use a HTTP-based front channel approach where an `<iframe>` is rendered in the user's browser as a mechanism to inform IdentityServer the user's session has ended (much like IdentityServer [supports](signout-http.html)).

To implement federated signout you must provide a signout endpoint (which would be registered with the upstream identity provider). When it's invoked it can inform all of IdentityServer's clients that the user has signed out. This is done by invoking `ProcessFederatedSignoutAsync` from the IdentityServer [OWIN environment extensions](owin.html). This will revoke the user's session cookies with IdentityServer and render the appropriate notification `<iframe>` to the client applications. Note: this requires your signout endpoint to run in the same pipeline as IdentityServer (meaning after).

The code below shows an example:

```csharp
public void Configuration(this IAppBuilder app)
{
   app.Map("/core", coreApp =>
   {
      var factory = new IdentityServerServiceFactory();

      // ...

      coreApp.UseIdentityServer(idsrvOptions);
      
      coreApp.Map("/signoutcleanup", cleanup =>
      {
         cleanup.Run(async ctx =>
         {
            await ctx.Environment.ProcessFederatedSignoutAsync();
         });
      });
   });
}
```
