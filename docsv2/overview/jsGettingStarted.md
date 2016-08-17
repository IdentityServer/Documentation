---
layout: docs-default
---

This tutorial walks you through the necessary steps to integrate IdentityServer in a JS application.
Since all the steps will be done on the client side, we'll use a JS library, [oidc-client-js](https://github.com/IdentityModel/oidc-client-js), to help with tasks like obtaining and validating tokens.

**TODO: link to final code when in the Samples repo.**

The walkthrough is split in 3 parts:

 - Authenticate against IdentityServer in the JS application
 - Make API calls from the JS application
 - Have a look at how to renew tokens, log out and check sessions

# Part 1 - Authentication against IdentityServer
This first part will focus on allowing us to authenticate in the JS application. To do so, we will create 2 projects; one for the JS application and one for IdentityServer.

## Create the JS application project
In Visual Studio, create an empty web application.

![create js app](https://cloud.githubusercontent.com/assets/6102639/12247759/9563a71a-b909-11e5-9823-6ba598b74bad.png)

Note the URL assigned to the project:

![js app url](https://cloud.githubusercontent.com/assets/6102639/12252652/4324b3b2-b92d-11e5-9641-772efe43c8e8.png)

## Create the IdentityServer project
In Visual Studio, create another empty web application for IdentityServer.

![create web app](https://cloud.githubusercontent.com/assets/6102639/12247585/bd0e33e4-b908-11e5-90bb-b6f3e9b2c060.png)

You can switch the project now to SSL using the properties window:

![set ssl](https://cloud.githubusercontent.com/assets/6102639/12252653/43288fc8-b92d-11e5-93eb-25821a64ceb2.png)

**Important**
Don't forget to update the start URL in your project properties so that it reflects the HTTPS url of the project.

## Adding IdentityServer
IdentityServer is based on OWIN/Katana and distributed as a NuGet package. To add it to the newly created web host, install the following two packages:

````
Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName IdentityServer
Install-Package IdentityServer3 -ProjectName IdentityServer
````

## Configuring IdentityServer - Clients
IdentityServer needs some information about the clients it is going to support, this can be easily achieved by supplying a collection of `Client` objects:

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client
            {
                Enabled = true,
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true
            }
        };
    }
}
```

A special setting here is the `AllowedCorsOrigins` property. This allows IdentityServer to only accept browser-based requests from registered URLs.
More on the `popup.html` later on.

**Remark** Right now the client has access to all scopes (via the `AllowAccessToAllScopes` setting). For production applications you would narrow that down to only the scopes it's expected to access with the `AllowedScopes` property.

## Configuring IdentityServer - Users
Next we will add some users to IdentityServer - again this can be accomplished by providing a simple C# class. You can retrieve user information from any data store and we provide out of the box support for ASP.NET Identity and MembershipReboot.

```csharp
public static class Users
{
    public static List<InMemoryUser> Get()
    {
        return new List<InMemoryUser>
        {
            new InMemoryUser
            {
                Username = "bob",
                Password = "secret",
                Subject = "1",

                Claims = new[]
                {
                    new Claim(Constants.ClaimTypes.GivenName, "Bob"),
                    new Claim(Constants.ClaimTypes.FamilyName, "Smith"),
                    new Claim(Constants.ClaimTypes.Email, "bob.smith@email.com")
                }
            }
        };
    }
}
```

## Configuring IdentityServer - Scopes
Finally we will add scopes to IdentityServer. For authentication we'll only put standard OIDC scopes. When we'll integrate API calls we'll create our own.

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile
        };
    }
}
```

## Adding Startup
IdentityServer is configured in the startup class. Here we provide information about the clients, users, scopes,
the signing certificate and some other configuration options.
In production you should load the signing certificate from the Windows certificate store or some other secured source.
In this sample we simply added it to the project as a file (you can download a test certificate from [here](https://github.com/identityserver/Thinktecture.IdentityServer3.Samples/tree/master/source/Certificates).
Add it to the project and set its `Copy to Output Directory` property to `Copy always`.


For info on how to load the certificate from Azure WebSites see [here](http://azure.microsoft.com/blog/2014/10/27/using-certificates-in-azure-websites-applications/).

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseIdentityServer(new IdentityServerOptions
        {
            SiteName = "Embedded IdentityServer",
            SigningCertificate = LoadCertificate(),

            Factory = new IdentityServerServiceFactory()
                .UseInMemoryUsers(Users.Get())
                .UseInMemoryClients(Clients.Get())
                .UseInMemoryScopes(Scopes.Get())
        });
    }

    private static X509Certificate2 LoadCertificate()
    {
        return new X509Certificate2(
            Path.Combine(AppDomain.CurrentDomain.BaseDirectory, @"bin\Config\idsrv3test.pfx"), "idsrv3test");
    }
}
```

At this point you have a fully functional IdentityServer and you can browse to the discovery endpoint to inspect the configuration:


![disco](https://cloud.githubusercontent.com/assets/6102639/12252651/431f61f0-b92d-11e5-9ca8-a49c3db5ea52.png)

## RAMMFAR
One last thing, please don't forget to add RAMMFAR to your web.config, otherwise some of our embedded assets will not be loaded correctly by IIS:

```xml
<system.webServer>
  <modules runAllManagedModulesForAllRequests="true" />
</system.webServer>
```

## JS Client - setup

We use several third-party libraries to support our application:

 - [jQuery](http://jquery.com)
 - [Bootstrap](http://getbootstrap.com)
 - [oidc-client-js](https://github.com/IdentityModel/oidc-client-js)

We are going to install them with [npm](https://www.npmjs.com/), the Node.js front-end package manager. If you don't have npm installed, you can follow [these instructions on the npm website](https://docs.npmjs.com/getting-started/installing-node).
Once npm is installed, open a command-line prompt in the `JsApplication` folder:

```sh
$ npm install jquery
$ npm install bootstrap
$ npm install oidc-client
```

By default, npm installs packages in the `node_modules` folder.

**Important** npm packages are usually not committed to source control. If you cloned the repository containing the final source code and want to restore the npm packages, open a
command-line prompt in the `JsApplication` folder and run `npm install` to restore packages.

We also create a basic `index.html` file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>JS Application</title>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="node_modules/bootstrap/dist/css/bootstrap.css" />
    <style>
        .main-container {
            padding-top: 70px;
        }

        pre:empty {
            display: none;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
        <div class="container">
            <div class="navbar-header">
                <a class="navbar-brand" href="#">JS Application</a>
            </div>
        </div>
    </nav>

    <div class="container main-container">
        <div class="row">
            <div class="col-xs-12">
                <ul class="list-inline list-unstyled requests">
                    <li><a href="index.html" class="btn btn-primary">Home</a></li>
                    <li><button type="button" class="btn btn-default js-login">Login</button></li>
                </ul>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-12">
                <div class="panel panel-default">
                    <div class="panel-heading">ID Token Contents</div>
                    <div class="panel-body">
                        <pre class="js-id-token"></pre>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script src="node_modules/jquery/dist/jquery.js"></script>
    <script src="node_modules/bootstrap/dist/js/bootstrap.js"></script>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
</body>
</html>
```

and a `popup.html` file:

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
</head>
<body>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
</body>
</html>
```

We have two HTML files because `oidc-client` can open a popup to show the login form to the user.

## JS Client - authentication

Now that we have everything we need, we can configure our login settings in `index.html` thanks to the `UserManager` JS class.

```js
// helper function to show data to the user
function display(selector, data) {
    if (data && typeof data === 'string') {
        data = JSON.parse(data);
    }
    if (data) {
        data = JSON.stringify(data, null, 2);
    }

    $(selector).text(data);
}

var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    response_type: 'id_token',
    scope: 'openid profile',

    filterProtocolClaims: true
};

var manager = new Oidc.UserManager(settings);
var user;

manager.events.addUserLoaded(function (loadedUser) {
    user = loadedUser;
    display('.js-user', user);
});

$('.js-login').click(function () {
    manager
        .signinPopup()
        .catch(function (error) {
            console.error('error while logging in through the popup', error);
        });
});
```

Let's go quickly through the settings:

 - `authority` is the base URL of our IdentityServer instance. This will allow `oidc-client` to query the metadata endpoint so it can validate the tokens
 - `client_id` is the id of the client we want to use when hitting the authorization endpoint
 - `popup_redirect_uri` is the redirect URL used when using the `signinPopup` method. If you prefer not having a popup and redirecting the user in the main window, you can use the `redirect_uri` property and the `signinRedirect` method
 - `response_type` defines in our case that we only expect an identity token back
 - `scope` defines the scopes the application asks for
 - `filterProtocolClaims` indicates to oidc-client if it has to filter some OIDC protocol claims from the response: `nonce`, `at_hash`, `iat`, `nbf`, `exp`, `aud`, `iss` and `idp`

We also handle clicks on the Login button to open the login page popup. The `signinPopup` returns a `Promise` which is resolved when the user data has been retrieved and validated.

This data is accessible via 2 ways:

 - as the resolved value of the underlying Promise
 - as the data associated with the `userLoaded` event

In our case, we added a handler to the `userLoaded` event by passing a callback function to the `events.addUserLoaded` method.
The data contains several properties like `id_token`, `scope` and `profile` that all contain different pieces of data.

We also have to configure `popup.html`:

```js
new Oidc.UserManager().signinPopupCallback();
```

Under the hoods, the instance of `UserManager` in the `index.html` page opens a popup and redirects it to the login page. When IdentityServer redirects the user to the popup page, the information is then passed back to the main page and the popup is automatically closed.

At this stage, you can login:

![login-popup](https://cloud.githubusercontent.com/assets/6102639/17659258/bae8530e-6314-11e6-8b1a-0570a26476cc.PNG)

![login-complete](https://cloud.githubusercontent.com/assets/6102639/17659287/e05f86f2-6314-11e6-941d-1fffe9b94678.PNG)

You can try and set the `filterProtocolClaims` property to `false` and see the additional claims being stored in the `profile` property.

## JS application - scopes

Remember we configured our user with an `email` claim? It doesn't show up in the identity token because the scopes the client asked for - `openid` and `profile` - don't contain this claim.
If you want to get the user's email, you'll have to ask for another scope which is called `email` by editing the `scopes` property of the `UserManager` settings.

In our case, the only modification we need to make is to let IdentityServer know that this scope exists in the `Scopes` class:

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile,
            // New scope
            StandardScopes.Email
        };
    }
}
```

We don't have to change anything in the client configuration because we specified it has access to all the scopes. In a realistic scenario, you want to give a client access to only the scopes it's expected to request, so this would require a change in the client configuration as well.

After this, we can see the `email` claim in the user profile:

![login-email](https://cloud.githubusercontent.com/assets/6102639/17659371/6a822c18-6315-11e6-9311-8bbf80e4bdc0.PNG)

# Part 2 - API call
In the second part, we'll see how you can call a protected API from the JS application.
We will need to get, along with the identity token, an access token from IdentityServer when we login and use it when calling the API.

## Create the API project
Create a new empty web application in Visual Studio.

![create api](https://cloud.githubusercontent.com/assets/6102639/12252754/251cd1f0-b92e-11e5-8f8f-469cfc2a0103.png)

## Configuring the API
For this example, we'll create a very simple API based on ASP.NET Web API.
To do so, install the following packages:

```
Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName Api
Install-Package Microsoft.Owin.Cors -ProjectName Api
Install-Package Microsoft.AspNet.WebApi.Owin -ProjectName Api
Install-Package IdentityServer3.AccessTokenValidation -ProjectName Api
```

**Important** The `IdentityServer3.AccessTokenValidation` package has an indirect dependency on `System.IdentityModel.Tokens.Jwt`.
At the time of writing, globally updating `Api` project NuGet packages brings down version `5.0.0` of `System.IdentityModel.Tokens.Jwt` which causes an error when starting the `Api` project:

![api-update-microsoft-identity-tokens](https://cloud.githubusercontent.com/assets/6102639/17659609/119a6be0-6317-11e6-9ca1-24889b813463.PNG)

The solution is to bring back an older compatible version of `System.IdentityModel.Tokens.Jwt`

```
Install-Package System.IdentityModel.Tokens.Jwt -ProjectName Api -Version 4.0.2.206221351
```

Let's now create a `Startup` class and build our OWIN/Katana pipeline.

```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        // Allow all origins
        app.UseCors(CorsOptions.AllowAll);

        // Wire token validation
        app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
        {
            Authority = "https://localhost:44300",

            // For access to the introspection endpoint
            ClientId = "api",
            ClientSecret = "api-secret",

            RequiredScopes = new[] { "api" }
        });

        // Wire Web API
        var httpConfiguration = new HttpConfiguration();
        httpConfiguration.MapHttpAttributeRoutes();
        httpConfiguration.Filters.Add(new AuthorizeAttribute());

        app.UseWebApi(httpConfiguration);
    }
}
```

This is all straight-forward, but let's have a closer look at what we use in our pipeline:

Since it is the JS application which will make the calls to the API, CORS must be enabled. In our case, we allow all origins to access it. Once again, in a real scenario, we would lock this down to allow only the expected origins.

We then use token validation provided by the `IdentityServer3.AccessTokenValidation` package. By setting the `Authority` property, the metadata document will be retrieved and used to configure the token validation settings.
Since version 2.2, IdentityServer implements the [introspection endpoint](../endpoints/introspection.html) to validate tokens. This endpoint requires scope authentication which makes it more secured than the traditional access token validation endpoint.

Finally, we add our Web API configuration. Note we use a global `AuthorizeAttribute` which makes every endpoint of the API only accessible to authenticated requests.

Let's now add a basic endpoint to our API:

```csharp
[Route("values")]
public class ValuesController : ApiController
{
    private static readonly Random _random = new Random();

    public IEnumerable<string> Get()
    {
        var random = new Random();

        return new[]
        {
            _random.Next(0, 10).ToString(),
            _random.Next(0, 10).ToString()
        };
    }
}
```

## Updating identityServer configuration
We introduced a new `api` scope which we have to register in IdentityServer. This is done by editing the `Scopes` class of the `identityServer` project:

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile,
            StandardScopes.Email,

            // New scope registration
            new Scope
            {
                Name = "api",

                DisplayName = "Access to API",
                Description = "This will grant you access to the API",

                ScopeSecrets = new List<Secret>
                {
                    new Secret("api-secret".Sha256())
                },

                Type = ScopeType.Resource
            }
        };
    }
}
```

The new scope is a resource scope which means it will end up in the access token. Once again, we don't need to allow the client to request this new scope in this example because of the special setting, but it will be a necessary step in a real scenario.

## Updating the JS application
We can now update the JS application settings so it will request the new `api` scope when logging in the user.

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    // We add `token` to specify we expect an access token too
    response_type: 'id_token token',
    // We add the new `api` scope to the list of requested scopes
    scope: 'openid profile email api',

    filterProtocolClaims: true
};
```

The modifications include:

 - a new panel to show the access token
 - an updated `response_type` to specify we want an access token back along with the identity token
 - the new `api` scope to be requested as part of the login request

The access token is exposed via the `access_token` property and its expiration via the `expires_at` property.
It is worth noting that `oidc-client` takes away a lot of pain by taking care of validating the tokens with the signing certificate, we don't have to write code.

After logging in, here's what we get:

![access-token](https://cloud.githubusercontent.com/assets/6102639/17659923/1ebf7a16-6319-11e6-99f7-33bef104d0e7.PNG)

## Calling the API

Now that we have an access token, we can include the call to the API:

```html
[...]
<div class="container main-container">
    <div class="row">
        <div class="col-xs-12">
            <ul class="list-inline list-unstyled requests">
                <li><a href="index.html" class="btn btn-primary">Home</a></li>
                <li><button type="button" class="btn btn-default js-login">Login</button></li>
                <!-- New button to trigger an API call -->
                <li><button type="button" class="btn btn-default js-call-api">Call API</button></li>
            </ul>
        </div>
    </div>

    <div class="row">
        <!-- Make the existing sections 6-column wide -->
        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">User data</div>
                <div class="panel-body">
                    <pre class="js-user"></pre>
                </div>
            </div>
        </div>

        <!-- And add a new one for the result of the API call -->
        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">API call result</div>
                <div class="panel-body">
                    <pre class="js-api-result"></pre>
                </div>
            </div>
        </div>
    </div>
</div>
```

```js
[...]
$('.js-call-api').click(function () {
    var headers = {};
    if (user && user.access_token) {
        headers['Authorization'] = 'Bearer ' + user.access_token;
    }

    $.ajax({
        url: 'http://localhost:60136/values',
        method: 'GET',
        dataType: 'json',
        headers: headers
    }).then(function (data) {
        display('.js-api-result', data);
    }).catch(function (error) {
        display('.js-api-result', {
            status: error.status,
            statusText: error.statusText,
            response: error.responseJSON
        });
    });
});
```

We now have a button which will trigger the API call along with another panel to show the call response.
Please note that the access token is passed in the `Authorization` header of the request.

Here's what we see if we call the API prior to login:

![api-without-access-token](https://cloud.githubusercontent.com/assets/6102639/17660059/2ce58a44-631a-11e6-8f68-c7a4edf85074.PNG)

And after login:

![api-with-access-token](https://cloud.githubusercontent.com/assets/6102639/17660060/2e898dfa-631a-11e6-9721-a0b98bd3fdcf.PNG)

In the first case, there was no access token, hence no `Authorization` header in the request, so the access token validation middleware did nothing. The request flowed through the API as unauthenticated, the global `AuthorizeAttribute` rejected it and responded with a `401 Unauthorized` error.
In the second case, the token validation middleware found the token in the `Authorization` header, passed it along to the introspection endpoint which flagged it as valid, and an identity was created with the claims it contained. The request, this time authenticated, flowed to Web API, the `AuthorizeAttribute` contrainsts were satisfied, and the endpoint was invoked.

# Part 3 - Renewing tokens, logging out and checking sessions

We now have a working JS application which logs in against IdentityServer and makes successful calls to a protected API.
But users will soon encounter issues when their access token expires and is rejected by the access token validation middleware on the API.
To work around that, we can setup `oidc-token-manager` to automatically renew the access token when it's about to expire, without any steps required for the user.

## Expired tokens

Let's first see how we can have a token expire on purpose. We have to reduce the lifetime of the access token.
This is a per-client setting, so we'll have to edit our `Clients` class in the IdentityServer project:

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client
            {
                Enabled = true,
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 10
            }
        };
    }
}
```

The access token lifetime, which is 1 hour by default, has been changed to 10 seconds.
What you'll experience if you login the JS application again is that you'll get the same `401 Unauthorized` error when you call the API 10 seconds after logging in.

## Renewing tokens

We are going to rely on a feature `oidc-client-js` gives us to renew the tokens.
Internally, the JS library keeps track of the expiration time of the access token and can request a new one by issuing a new authorization request to IdentityServer.
This will be invisible to the user as the [`prompt`](../endpoints/authorization.html) setting, which will be set to `none`, prevents the user from having to log in or give his consent while he has a valid session.
IdentityServer will return a new access token which will replace the one that is about to expire.

There are several settings related to access token expiration and renewal:

 - The [`accessTokenExpiring`](https://github.com/IdentityModel/oidc-client-js/wiki#events) event will be fired when the access token is about to expire
 - The [`accessTokenExpiringNotificationTime`](https://github.com/IdentityModel/oidc-client-js/wiki#configuration) can be used to tweak how far before the token expires the `accessTokenExpiring` event is fired. The default value is `60` seconds
 - Another setting is named `automaticSilentRenew` which instructs the library to automatically renew the access token when it's about to expire
 - Finally, the `silent_redirect_uri` setting needs to be configured so the library can specify it as a return URL when trying to get a new token

Here is how `oidc-client-js` handles automatic token renewal.
When the token is about to expire, a dynamic hidden `iframe` will be created.
In this `iframe` a new authorization request will be made to IdentityServer. If the request succeeds, identityServer will redirect the `iframe` to the specified silent redirect URL, in which a piece of JS code will update the user information so that the main window gets access to it/

Let's make some modifications in our configuration to take advantage of these capabilities.

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',
    // Add the slient renew redirect URL
    silent_redirect_uri: 'http://localhost:56668/silent-renew.html'

    response_type: 'id_token token',
    scope: 'openid profile email api',

    // Add expiration nofitication time
    accessTokenExpiringNotificationTime: 4,
    // Setup to renew token access automatically
    automaticSilentRenew: true,

    filterProtocolClaims: true
};
```

Since we specified a new page as the `silent_redirect_uri`, we have to create that page

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <meta charset="utf-8" />
</head>
<body>
    <script src="node_modules/oidc-client/dist/oidc-client.js"></script>
    <script>
        new Oidc.UserManager().signinSilentCallback();
    </script>
</body>
</html>
```

The second step is to let IdentityServer know that it is OK to redirect the user to that new page after the authentication is successful:

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client
            {
                Enabled = true,
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html",
                    // The new page is a valid redirect page after login
                    "http://localhost:56668/silent-renew.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 70
            }
        };
    }
}
```

When the renewal is successful, the `UserManager` will raise a `userLoaded` event.
Since we already handle this event, the updated data will automatically be captured and displayed on the UI.

When it fails, it will raise a `silentRenewError` event, which we can subscribe to so we know when something went wrong

```js
manager.events.addSilentRenewError(function (error) {
    console.error('error while renewing the access token', error);
});
```

We updated the access token lifetime to 10 seconds and instructed `oidc-client-js` to renew the token 4 seconds before it expires.
So now, after logging in, we can see that every 6 seconds we get a fresh access token from IdentityServer.

## Logging out

Logging out of a JS application has a different meaning than from a server-side application, because if you refresh the main page, you will lose the tokens and will have to login again.
But when the login popup opens, it could be that you still have a valid session cookie for the IdentityServer web application. It could then be possible that the popup doesn't prompt you for your credentials and close itself. This is similar to when the token manager silently renews the token.

Logging out here means logging out of IdentityServer so that, next time you try to login from an IdentityServer-protected application, you will have to enter your credentials again.

The process here is simple, we just need a logout button that calls the `redirectForLogout` method of the `OidcTokenManager` instance. We also need to let IdentityServer know that the specified post-logout redirect URL is valid:

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client
            {
                Enabled = true,
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "http://localhost:56668/popup.html",
                    "http://localhost:56668/silent-renew.html"
                },

                // Valid URLs after logging out
                PostLogoutRedirectUris = new List<string>
                {
                    "http://localhost:56668/index.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "http://localhost:56668"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 70
            }
        };
    }
```

```html
[...]
<div class="row">
    <div class="col-xs-12">
        <ul class="list-inline list-unstyled requests">
            <li><a href="index.html" class="btn btn-primary">Home</a></li>
            <li><button type="button" class="btn btn-default js-login">Login</button></li>
            <li><button type="button" class="btn btn-default js-call-api">Call API</button></li>
            <!-- New logout button -->
            <li><button type="button" class="btn btn-danger js-logout">Logout</button></li>
        </ul>
    </div>
</div>
```

```js
[...]
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    silent_renew: true,
    silent_redirect_uri: 'http://localhost:56668/silent-renew.html',

    response_type: 'id_token token',
    scope: 'openid profile email api',

    // post-logout URL
    post_logout_redirect_uri: 'http://localhost:56668/index.html',

    filter_protocol_claims: true
};
[...]
$('.js-logout').click(function () {
    manager.redirectForLogout();
});
```

When clicking the `Logout` button, the user will be redirected to IdentityServer so that the session cookie is cleared.

[logout](https://cloud.githubusercontent.com/assets/6102639/12256384/d9de8df0-b950-11e5-91b2-650a0a749a7f.png)

_Please note the screenshot above shows a page served by IdentityServer, not the JS application_

## Check session

The session in our JS application starts when the identity token we get back from IdentityServer is validated.
IdentityServer itself supports session management so it returns, in the authorization response, a value named `session_state`.

In some cases, we might be interested to know if the user ended his session on IdentityServer, for example by logging out of another application which in turned logged him out of IdentityServer.
The idea is to compute again the `session_state` value. If it's equal to the one IdentityServer sent, this means the session state is unchanged, so the user is still logged in. It it's different, something changed, possibly the user logged out. In this case it's advised to issue a silent authorization request, with `prompt=none`. If it succeeds, we get a new identity token, and it means the session on the IdentityServer side is still valid. If it fails, the user has logged out, and we have to ask him to log in again.

Unfortunately, the JS application on its own cannot compute the `session_state` value because it depends on the IdentityServer session cookie value which it doesn't have access to.
The design of the [specification](https://openid.net/specs/openid-connect-session-1_0.html) requires to load, in a hidden `iframe`, the checksession endpoint from IdentityServer. The JS application and this `iframe` can then communicate with the [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) function.

### The checksession endpoint

This endpoint serves a simple page which listens to messages sent with `postMessage`. The data passed in the message is used to compute the session state hash. If it matches the one sent by IdentityServer, the page sends a message back to the calling window with the value `unchanged`. It it doesn't, it sends back `changed`. If something goes wrong, it sends `error`.

### Prerequisites

Before going further, we have to make a slight modification.
If the checksession `iframe` - loaded from IdentityServer, thus over HTTPS - is loaded from a page served over HTTP, we will run into issues.
The solution is to serve our JS application over HTTPS, too.

Here are the several steps to take:

First, enable SSL for the `JsApplication` project, and change the URL in the project properties, as we did for the `IdentityServer` project:

![idsrv app url](https://cloud.githubusercontent.com/assets/6102639/12341948/a1c58718-bb79-11e5-9b8d-701e20d029b7.png)

Then, in the `Clients` class of the `IdentityServer` project, replace all the URLs with the new ones:

```csharp
public static class Clients
{
    public static IEnumerable<Client> Get()
    {
        return new[]
        {
            new Client
            {
                Enabled = true,
                ClientName = "JS Client",
                ClientId = "js",
                Flow = Flows.Implicit,

                RedirectUris = new List<string>
                {
                    "https://localhost:44301/popup.html",
                    "https://localhost:44301/silent-renew.html"
                },

                PostLogoutRedirectUris = new List<string>
                {
                    "https://localhost:44301/index.html"
                },

                AllowedCorsOrigins = new List<string>
                {
                    "https://localhost:44301"
                },

                AllowAccessToAllScopes = true,
                AccessTokenLifetime = 70
            }
        };
    }
}
```

Also replace the URLs in the token manager settings in the `index.html` file of the JS application:

```js
[...]
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'https://localhost:44301/popup.html',

    silent_renew: true,
    silent_redirect_uri: 'https://localhost:44301/silent-renew.html',

    response_type: 'id_token token',
    scope: 'openid profile email api',

    post_logout_redirect_uri: 'https://localhost:44301/index.html',

    filter_protocol_claims: true
};
```

Since you cannot access an HTTP resource from an HTTPS resource, we have to make the API available over HTTPS, too. Enable SSL as we did for the other projects, and update the URL in `index.html`:

```js
$('.js-call-api').click(function () {
    var headers = {};
    if (manager.access_token) {
        headers['Authorization'] = 'Bearer ' + manager.access_token;
    }

    $.ajax({
        url: 'https://localhost:44302/values',
        method: 'GET',
        dataType: 'json',
        headers: headers
    }).then(function (data) {
        display('.js-api-result', data);
    }).fail(function (error) {
        display('.js-api-result', {
            status: error.status,
            statusText: error.statusText,
            response: error.responseJSON
        });
    });
});
```

### Building the session check feature

Now, create a `check-session.html` file with the following contents:

```html
<!DOCTYPE html>
<html>
<head>
    <title>RP Check Session IFrame</title>
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
</head>
<body>
    <iframe id="op"></iframe>
    <script>
        if (window.location.hash) {
            // 1 - getting parameters
            var hash = window.location.hash.substr(1);
            var params = hash.split('&').reduce(function (curr, item) {
                var parts = item.split('=');
                curr[parts[0].trim()] = parts[1].trim();
                return curr;
            }, {});

            // 2 - parameters check
            if (!params.session_state) {
                console.error('No session_state passed to RP session frame');
            }
            else if (!params.check_session_iframe) {
                console.error('No check_session_iframe passed to RP session frame');
            }
            else if (!params.client_id) {
                console.error('No client_id passed to RP session frame');
            }
            else {
                var idSrvOrigin = 'https://localhost:44300';

                var frame = document.getElementById('op');
                frame.onload = function () {
                    // 3 - posting messages
                    var op = frame.contentWindow;
                    var timer = window.setInterval(function () {
                        op.postMessage(params.client_id + ' ' + params.session_state, idSrvOrigin);
                    }, 2000);

                    // 4 - receiving messages
                    window.addEventListener('message', function (e) {
                        if (e.origin === idSrvOrigin) {
                            console.log('session state', e.data);
                            if (e.data === 'changed' || e.data === 'error') {
                                window.clearInterval(timer);
                            }

                            if (e.data === 'changed') {
                                window.parent.postMessage('changed', 'https://localhost:44301');
                            }
                        }
                    });
                };

                // 5 - loading the iframe
                frame.src = params.check_session_iframe;
            }
        }
    </script>
</body>
</html>
```

An important notion is to get that this page will be loaded by `index.html` in an `iframe`.
As you can see, it itself contains an `iframe`, in which we'll load the checksession page from IdentityServer.
In this section, we'll differentiate them by calling them JS application iframe and IdentityServer iframe.

Now, let's go through the different parts:

The first code block parses the parameters passed in the URL after the hash sign. They are structured as in a query string, that is `key=value`.
The result is stored in the `params` variable.

The second part makes sure all the expected parameters were provided. We need 3 of them:

 - The `session_state` returned by IdentityServer
 - The URL of the checksession endpoint so we know where to load the `iframe` from
 - The client id, which is needed by the checksession endpoint to compute the session state hash

Part #3 will be executed when the checkssion `iframe` is loaded. The idea here is to poll the checksession `iframe` by regularly sending messages with the `postMessage` function.

For each sent message, a response message will be received which is what the fourth part takes care of.
As a security measure, we make sure the message comes from the IdentityServer iframe since potentially any window could send a message.
If the value we get back from IdentityServer is `changed`, we pass the message through by sending a message to the `index.html` file.

The fifth and final part loads the IdentityServer iframe.

### Wiring it in `index.html`

The only missing piece is now to integrate what we built in the main page:

```html
[...]
<!-- Will be used to load check-session.html -->
<iframe id="rp" style="display:none"></iframe>

<script src="bower_components/jquery/dist/jquery.js"></script>
<script src="bower_components/bootstrap/dist/js/bootstrap.js"></script>
<script src="bower_components/oidc-token-manager/dist/oidc-token-manager.js"></script>
```

```js
[...]
function checkSessionState() {
    manager.oidcClient.loadMetadataAsync().then(function (meta) {
        if (meta.check_session_iframe && manager.session_state) {
            document.getElementById('rp').src = 'check-session.html#' +
                'session_state=' + manager.session_state +
                '&check_session_iframe=' + meta.check_session_iframe +
                '&client_id=' + settings.client_id;
        }
        else {
            document.getElementById('rp').src = 'about:blank';
        }
    });

    window.onmessage = function (e) {
        if (e.origin === 'https://localhost:44301' && e.data === 'changed') {
            manager.removeToken();
            manager.renewTokenSilentAsync().then(function () {
                // Session state changed but we managed to silently get a new identity token, everything's fine
                console.log('renewTokenSilentAsync success');
            }, function () {
                // Here we couldn't get a new identity token, we have to ask the user to log in again
                console.log('renewTokenSilentAsync failed');
            });
        }
    }
}
[...]
$('.js-login').click(function () {
    manager.openPopupForTokenAsync()
        .then(function () {
            display('.js-id-token', manager.profile);

            // Load the iframe and start listening to messages
            checkSessionState();

            // Removed
            // display('.js-access-token', { access_token: manager.access_token, expires_in: manager.expires_in });
        }, function (error) {
            console.error(error);
        });
});
```

When we initially get a token back from IdentityServer, we now call the `checkSessionState` function.
It requests IdentityServer metadata endpoint to make sure the checksession endpoint is available since it could have been disabled.
If it is, the `check-session.html` is loaded in a hidden `iframe`, passing along the parameters it needs.
A listener for messages is then created. Here again, we want to make sure they come from the `check-session.html` page.

As discussed earlier, in the case where the session state has changed, a silent authorization request is sent.
If it succeeds, it means the session on IdentityServer is still valid and we got a new identity token.
If it fails, the user must have logged out, so we have to ask him to log in again.

We can simulate that by opening two tabs to the `index.html` file, logging in both of them. Inspecting `manager.session_state` in the console will give the same value.
If we log out from one tab, we are redirected to IdentityServer where the session cookie will be cleared. Next time the session state is computed in the IdentityServer iframe of the other tab,
it will detect that it has changed.