---
layout: docs-default
---

# Migrating from 1.x to 2.0

IdentityServer 2.0.0 changed some client semantics, so there is a necessary EF migration. Unfortunately the EF migration tool does not understand all of these semantics, but fortunately this is not that difficult with some manual tweaks. Here is a link to what the final migration should look like:

[https://github.com/IdentityServer/IdentityServer3.Samples/blob/dev/source/EntityFramework/SelfHost/Migrations/ClientConfiguration/201504070205330_v2_0_0.cs](https://github.com/IdentityServer/IdentityServer3.Samples/blob/dev/source/EntityFramework/SelfHost/Migrations/ClientConfiguration/201504070205330_v2_0_0.cs)
