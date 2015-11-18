---
layout: docs-default
---

#Signout Support

IdentityServer supports two styles of signout notifications. 
These styles coorespond to two (of the three) different OpenID Connection session
management specifications: 
the [session management](https://openid.net/specs/openid-connect-session-1_0.html) 
and the [HTTP-based logout](https://openid.net/specs/openid-connect-logout-1_0.html) 
specifications. 
One is designed for JavaScript-based client-side applications, 
and the other is designed for server-side web applications (ASP.NET, etc). 

More info for support in IdentityServer for each can be found here:

* [JavaScript-based client-side](signout-session.html)
* [HTTP-based server-side](signout-http.html)
