---
layout: docs-default
---

## Custom View Service

**The sample for this sub-topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/CustomViewService)**

If the hosting application requires complete control over the views (HTML, CSS, JavaScript, etc.) then it can implement the `IViewService` to control all of the markup rendered for the views.  The custom view service would then be registered with the `ViewService` property of the `IdentityServerServiceFactory`.

The methods of the `IViewService` interface each are expected to produce a `Stream` that contains the UTF8 encoded markup to be displayed for the various views (login, consent, etc.). These methods all accept as parameters a model that is specific to the view being rendered (e.g. a `LoginViewModel`, or a `ConsentViewModel`). This model provides contextual information that will most likely be needed to be presented to the user (for example the client and scopes on the consent view, or the error message on the error view, or the URL to submit credentials to login).

Most views will need to make requests back to various endpoints within IdentityServer. These GET and POST requests are required to contain the same inputs that the default views send to the server. The URLs are included in the various models.

### Anti Cross-Site Request Forgery (Anti-XSRF)

The views that POST back to the server are required to include an anti-xsrf token in the form-URL-encoded request body. Each pertinent model contains a `AntiForgeryTokenViewModel` object that has the `Name` and `Value` of the parameter that is expected. Custom views must include these when submitting the user's input data.
