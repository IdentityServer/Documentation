---
layout: docs-default
---

# Terminology

The specs, documentation and object model use a certain terminology that you should be aware of.

![modern application architecture]({{ site.baseurl }}/assets/images/terminology.png)

### OpenID Connect Provider
IdentityServer is an OpenID Connect provider - it implements the OpenID Connect protocol (and OAuth2 as well).

Different literature uses different terms for the same role - you probably also find security token service,
identity provider, authorization server, IP-STS and more.

But they are in a nutshell all the same: a piece of software that issues security tokens to clients.

IdentityServer has a number of jobs and features - including:

* authenticate users using a local account store or via an external identity provider

* provide session management and single sign-on

* manage and authenticate clients

* issue identity and access tokens to clients

* validate tokens for clients


### User
A user is a human

## Client

## Scope

## Authentication/Token Request

## Identity Token

## Access Token