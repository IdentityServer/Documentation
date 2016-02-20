---
layout: docs-default
---

## Default View Service

**The sample for this topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/DefaultViewService)**

The default implementation of the view service used by IdentityServer is the `DefaultViewService`. The various assets (HTML, JavaScript, CSS, and fonts) that comprise the views are served up from embedded resources within the IdentityServer assembly.

The `DefaultViewService` allows for some amount of customization.

### CSS and JavaScript customization

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

### HTML customization

The views in the `DefaultViewService` use AngularJS to perform dynamic rendering client-side. The rendering is driven by a view model that is included in each view. Each view has a different view model with the pertinent data for the view.

The `DefaultViewService` does allow for the HTML to be customized, but the assumption is that the same approach will be used for the dynamic rendering of the HTML. If this is not desirable, then consider [implementing a custom view service](customViewService.html).

Also, many of the views have links or forms that make requests back into IdentityServer. Any customizations to the views must be able to recreate the same requests into IdentityServer. In other words, a custom view can not change how the server is expecteding to be invoked from the view.

When rendering its views, the `DefaultViewService` uses a templating mechanism (similar to layout templates in MVC). There is a single shared "layout" view, and then there are separate "partial" views (one for each page that needs to be rendered) that contain the contents to be rendered inside of the "layout". When a view is rendered these two are merged together on the server to emit a single HTML document.

You can see the default layout and partial views [here](https://github.com/IdentityServer/IdentityServer3/tree/master/source/Core/Services/DefaultViewService/PageAssets).

#### Replacing entire views

The default views can be replaced by creating HTML files within an `templates` folder within the hosting applications base directory. The files contained inside are located by name matching the view being rendered (e.g. the login view will look for a file named `login.html`). If these files are present then they are responsible for rendering the entirety of the HTML for that view.

#### Replacing partial views

When customizing the HTML, rather than replacing the entire HTML document you can replace just the partial view. To replace just the partial view the file located in the `templates` folder simply needs to be distinguished by prefixing the name with an underscore (e.g. `_login.html`). This will then use the default layout template, but use the custom partial view. It will be merged into the layout template to render the combined HTML to the browser.

In addition to being able to replace the partial views, it's also possible to replace the default layout template itself. This can be done by creating a file named `_layout.html` in the `templates` folder. The `DefaultViewService` will then use whatever combination of custom layout or partial views discovered on the file system to merge with the default embedded assets to render the requested view.

#### Custom views location (Added in v2.4)

By default, the custom views are located in a directory named `templates` within the hosting applications base directory. This can be customized by specifying the `CustomViewDirectory` property on the `DefaultViewServiceOptions`. This must be the full filesystem path to the directory.

For example:

```csharp
var viewOptions = new DefaultViewServiceOptions();
viewOptions.CustomViewDirectory = @"C:\IdentityServerTemplates";

var factory = new IdentityServerServiceFactory();
factory.ConfigureDefaultViewService(viewOptions);
```

#### Caching

The custom views will be cached in-memory by default, so if the files are changed then it will require an application restart to load any updated HTML. This behavior can be disabled by setting the `CacheViews` property to `false` on the `DefaultViewServiceOptions` described earlier.

#### Custom view loader

Finally, if the `templates` folder on the file system is not desirable, then you can implement your own storage for the custom views by implmeneting the `IViewLoader` interface. This is configured as a `Registration<IViewLoader>
    ` on the  `DefaultViewServiceOptions`.

### Adding custom data to the rendered view

It is possible that custom views need to render custom data. All of the view models used by the  `DefaultViewService` provide a `object` property called `Custom`. This is simply rendered into the client-side view model and is available for use in the AngularJS templates. 
    
To provide `Custom` data on the view models, it will be necessary to derive from the `DefaultViewService` and override the appropriate methods for the views where the custom data needs to be rendered. 

For example, to add a custom message to the login page:

```csharp
public class CustomViewService : DefaultViewService
{
    public CustomViewService(DefaultViewServiceOptions config, IViewLoader viewLoader)
        : base(config, viewLoader)
    {
    }

    public override Task<Stream> Login(LoginViewModel model, SignInMessage message)
    {
        model.Custom = new {
            customMessage = "Hello World!"
        };

        return base.Login(model, message);
    }
}
```

Then in the `_login.html` template, this custom markup can be added to access the `customMessage` property:

```html
<div ng-show="model.custom.customMessage">
    <h2>{ { model.custom.customMessage } }</h2>
</div>
```

Finally, the `CustomViewService` must be registered with IdentityServer:

```csharp
var factory = new IdentityServerServiceFactory();
factory.ViewService = new DefaultViewServiceRegistration<CustomViewService>();
```
