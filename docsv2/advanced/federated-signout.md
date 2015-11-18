---
layout: docs-default
---

#Federated Signout

IdentityServer supports the ability to federate with external identity providers. When a user signs out of an upstream identity provider, depending upon the protocol used, it might be possible to receive a notification when the user has signed out. This would then allow IdentityServer to notify its clients so they can also sign the user out.

A common approach in both WS-Federation and OIDC providers is to use a HTTP-based front channel approach where an `<iframe>` is rendered in the user's browser as a mechanism to inform IdentityServer the user's session has ended (much like IdentityServer [supports](signout-http.html)).

When federating with your upstream identity provider, if a HTTP-based signout is supported, then you can provide an signout endpoint that in turn will inform all of IdentityServer's clients that the user has signed out. This requires you to implement an endpoint and then invoke `ProcessFederatedSignoutAsync` (via the IdentityServer OWIN environment extensions) that will render the appropriate notification `<iframe>`. Note: this requires your signout endpoint to run in the same pipeline as IdentityServer (meaning after).

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
