---
layout: docs-default
---

# Terminology

The specs, documentation and object model use a certain terminology that you should be aware of.

![modern application architecture]({{ site.baseurl }}/assets/images/terminology.png)

## OpenID Connect Provider (OP)
IdentityServer is an OpenID Connect provider - it implements the OpenID Connect protocol (and OAuth2 as well).

Different literature uses different terms for the same role - you probably also find security token service,
identity provider, authorization server, IP-STS and more.

But they are in a nutshell all the same: a piece of software that issues security tokens to clients.

IdentityServer has a number of jobs and features - including:

* authenticate users using a local account store or via an external identity provider

* provide session management and single sign-on

* manage and authenticate clients

* issue identity and access tokens to clients

* validate tokens

## Client
A client is a piece of software that requests tokens from IdentityServer - either for authenticating a user or
for accessing a resource.

Examples for clients are web applications, native mobile or desktop applications, SPAs, server processes etc.

## User
A user is a human that is using a client.

## Scope
Scopes are identifiers for resources that a client wants to access. This identifier is sent to the OP during an
authentication or token request.

They come in two flavours.

### Identity scopes
Identity information (aka claims) about a user, e.g. his name or email address is modeled as a scope in OpenID Connect.

There is e.g. a scope called `profile` that includes first name, last name, preferred username, gender, profile picture and more.
You can read about the standard scopes [here](http://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) and you can create your own scopes in IdentityServer to model your own requirements.


### Resource scopes

## Authentication/Token Request

## Identity Token

## Access Token