---
layout: docs-default
---

Most modern applications look more or less like this:

![modern application architecture]({{ site.url}}{{ site.baseurl }}/docs/overview/images/appArch.png)

The typical interactions are:
-   Browsers communicate with web applications
-  Web applications communicate with web APIs (sometimes on their own, sometimes on behalf of a user)
-  Browser-based applications communicate with web APIs
-  Native applications communicate with web APIs
-  Server-based applications communicate with web APIs
-  Web APIs communicate with web APIs (sometimes on their own, sometimes on behalf of a user)

Typically each layer (front-end, middle-tier and back-end) has to protect some resources and implement authentication and authorization – and quite typically against the same user store.

This is why we don’t implement these fundamental security functions in the business applications/endpoints itself, but rather outsource that critical functionality to a security token service.

This leads to the following security architecture and usage of protocols:

![security architecture and protocols]({{ site.url }}{{ site.baseurl }}/docs/overview/images/protocols.png)

This divides the security world into two parts.

**Authentication**

Authentication is needed when an application needs to know about the identity of the current user. Typically these applications manages data on behalf of that user and needs to make sure that the use can only access the data he is allowed to. The most common example for that are (classic) web applications – but also native and JS-based applications have need for authentication.

The most common authentication protocols are SAML2p, WS-Federation and OpenID Connect – SAML2p being the most popular and widely deployed one.

OpenID Connect is the newest of the three, but is generally considered being the future because it has the most potential for modern applications. It was being built for mobile applications scenarios right from the start and is designed to be API friendly.

**API Access**

Applications have two fundamental ways how they communicate with APIs – using the application identity, or delegating the user’s identity. Sometimes both ways need to be combined.

OAuth2 is a protocol that allows applications to request access tokens from a security token service and use them to communicate with APIs. This reduces complexity on both the client applications as well as the APIs since authentication and authorization can be centralized.

**OpenID Connect and OAuth2 – better together**

OpenID Connect and OAuth2 are very similar – in fact OpenID Connect is an extensions on top of OAuth2. This means that you can combine the two fundamental security concerns – authentication and API access into a single protocol – and often a single round trip to the security token service.

This is why we believe that the combination of OpenID Connect and OAuth2 is the best approach to secure modern applications for the foreseeable future. IdentityServer v3 is an implementation of these two protocols and is highly optimized to solve the typical security problems of today’s mobile, native and web applications.