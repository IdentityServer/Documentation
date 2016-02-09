---
layout: docs-default
---

# Session management for client-side JavaScript-based applications

The [Session management](https://openid.net/specs/openid-connect-session-1_0.html) specification defines a mechanism for an OpenID Connect provider to inform client-side JavaScript-based applications that a user has signed out. 

The mechanism defined in the specification involves the JavaScript application opening an `<iframe>` to the OpenID Connect provider's "check\_session\_iframe" (whose value should be accessible from the metadata endpoint). This `<iframe>` (given that it is from the OP's origin) can access the cookies managed by the OP and can detect when the user's login session has changed (meaning the user has signed out, or has signed in as another user). 

The JavaScript client application can perodically use `postMessage` to the `<iframe>` to ask if there have been any changes to the user's session. The `<iframe>` will reply with either `"changed"` or `"unchanged"`. If "changed" is the response, the JavaScript application then knows that the user's session has ended (or changed in some way) and can then perform any cleanup necessary.


**To use this technique for signout notification, consult the sample JavaScript application [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/Clients/JavaScriptImplicitClient).**
