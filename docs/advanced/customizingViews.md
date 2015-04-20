---
layout: docs-default
---

## Customizing Views

IdentityServer3 displays various “views” to the user. IdentityServer requires views for login, logout prompt, logged out, consent, client permissions, and errors. These views are simply web pages displayed in the browser. To obtain the markup for these views, IdentityServer defines the `IViewService` interface. The view service is one of the optional extensibility points in IdentityServer.

### Default View Service

**The sample for this sub-topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/EmbeddedAssetsViewService)**

The default implementation of the view service used by IdentityServer is the `DefaultViewService`. The various assets (HTML, JavaScript, CSS, and fonts) that comprise the views are served up from embedded resources within the IdentityServer assembly.

The `DefaultViewService` allows for some amount of customization.

#### CSS and Script customization

One simple approach to customization is that the hosting application can provide a list of either CSS and/or JavaScript files to include in the default web pages. This allows for branding without the need to completely replace the assets themselves.

The `DefaultViewServiceOptions` class is used to indicate these CSS and/or JavaScript files via the `Stylesheets` and `Scripts` lists:

```csharp
var viewOptions = new DefaultViewServiceOptions();
viewOptions.Stylesheets.Add("/Content/Site.css");
```
The paths passed to `Add` can either be relative to IdentityServer’s base path by prefixing the path with a “~” (such as “~/path/file.css”), or the path can be host-relative by prefixing the path with a “/” (such as “/path/file.css”). Absolute URLs are  supported as long as they are added to IdentityServer's [CSP options](csp.html).

To then use the `DefaultViewServiceOptions` there is a `ConfigureDefaultViewService` extension method on the  `IdentityServerServiceFactory`. This helper method uses the dependency injection system to register the `DefaultViewService` and its dependencies based upon the options:

```csharp
var viewOptions = new DefaultViewServiceOptions();
viewOptions.Stylesheets.Add("/Content/Site.css");

var factory = new IdentityServerServiceFactory();
factory.ConfigureDefaultViewService(viewOptions);
```

#### HTML customization

##### Replacing entire views

The `DefaultViewService` does allow for the HTML to be customized. The default views can be "overridden" by creating HTML files within an `assets` folder within the hosting applications base directory. The files contained inside are located by name matching the view being rendered (e.g. the login view will look for a file named `login.html`). If these files are present then they are responsible for rendeing the entirety of the HTML for that view.

##### Replacing partial views

When rendering its default views, the `DefaultViewService` uses a templating mechanism (similar to layout templates in MVC). There is a single shared "layout" view, and then there are separate "partial" views (one for each page that needs to be rendered) that conatin the contents to be rendered inside of the "layout". When a view is rendered these two are merged together on the server to emit a single HTML document.

This point of this discussion is that when customizing the HTML, rather than replacing the entire HTML document you can replace just the partial view. To replace just the partial view the file located in the `assets` folder simply needs to be distinguised by prefixing the name with an underscope (e.g. `_login.html`). This will then use the default layout template, but use the custom partial view. It will be merged into the layout template to render the combined HTML to the browser.

In addition to being able to replace the partial views, it's also possible to replace the default layout template itself. This can be done by creating a file named `_layout.html` in the `assets` folder. The `DefaultViewService` will then use whatever combination of custom layout or partial views discovered on the file system to merge with the default embedded assets to render the requested view.

##### Caching

The custom views will be cached in-memory by default, so if the files are changed then it will require an application restart to load any updated HTML. This behavior can bedisabled by setting the `CacheViews` property to `false` on the `DefaultViewServiceOptions` described earlier.

##### Custom view loader

Finally, if the `assets` folder on the file system is not desirable, then you can implement your own storage for the custom views by implmeneting the `IViewLoader` interface. This is configured as a `Registration<IViewLoader>` on the  `DefaultViewServiceOptions`.

### Custom View Service

**The sample for this sub-topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/CustomViewService)**

If the hosting application requires complete control over the views (HTML, CSS, JavaScript, etc.) then it can implement the `IViewService` to control all of the markup rendered for the views.  The custom view service would then be registered with the `ViewService` property of the `IdentityServerServiceFactory`.

The methods of the `IViewService` interface each are expected to produce a `Stream` that contains the UTF8 encoded markup to be displayed for the various views (login, consent, etc.). These methods all accept as parameters a model that is specific to the view being rendered (e.g. a `LoginViewModel`, or a `ConsentViewModel`). This model provides contextual information that will most likely be needed to be presented to the user (for example the client and scopes on the consent view, or the error message on the error view, or the URL to submit credentials to login).

Most views will need to make requests back to various endpoints within IdentityServer. These GET and POST requests are required to contain the same inputs that the default views send to the server. The URLs are included in the various models.

#### Anti Cross-Site Request Forgery (Anti-XSRF)

The views that POST back to the server are required to include an anti-xsrf token in the form-URL-encoded request body. Each pertinent model contains a `AntiForgeryTokenViewModel` object that has the `Name` and `Value` of the parameter that is expected. Custom views must include these when submitting the user's input data.
