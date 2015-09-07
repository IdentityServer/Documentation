---
layout: docs-default
---

## Customizing Views

IdentityServer3 displays various “views” to the user. IdentityServer requires views for login, logout prompt, logged out, consent, client permissions, and errors. These views are simply web pages displayed in the browser. To obtain the markup for these views, IdentityServer defines the `IViewService` interface. IdentityServer provides default views (via the `DefaultViewService` implementation). 

The views in IdentityServer can be customized in one of two ways: 1) Customize the HTML templates provided by the  `DefaultViewService`, or if more control is needed 2) define a custom `IViewService`.

* [Customizing the `DefaultViewService`](DefaultViewService.html)
* [Creating a custom `IViewService`](customViewService.html)
