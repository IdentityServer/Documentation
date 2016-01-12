---
layout: docs-default
---

This tutorial walks you through the necessary steps to integrate IdentityServer in a JS application.  
Since all the steps will be done on the client side, we'll use a JS library, [oidc-token-manager](https://github.com/IdentityModel/oidc-token-manager) to help with tasks like getting and validating tokens.

We'll split the walkthrough in 3 parts:

 - Authenticate against IdentityServer in the JS application
 - Make API calls from the JS application
 - Have a look at how to renew tokens, check sessions and log out

# Part 1 - Authentication against IdentityServer
This first part will focus on allowing us to authenticate in the JS application.

## Create the JS application project
In Visual Studio, create an empty web application and set authentication to "No authentication".

![create js app](https://cloud.githubusercontent.com/assets/6102639/12247759/9563a71a-b909-11e5-9823-6ba598b74bad.png)

Note the URL assigned to the project:

![js app url](https://cloud.githubusercontent.com/assets/6102639/12252652/4324b3b2-b92d-11e5-9641-772efe43c8e8.png)

## Create the IdentityServer project
In Visual Studio, create an empty web application and set authentication to "No authentication".

![create web app](https://cloud.githubusercontent.com/assets/6102639/12247585/bd0e33e4-b908-11e5-90bb-b6f3e9b2c060.png)

You can switch the project now to SSL using the properties window:

![set ssl](https://cloud.githubusercontent.com/assets/6102639/12252653/43288fc8-b92d-11e5-93eb-25821a64ceb2.png)

**Important**
Don't forget to update the start URL in your project properties so that it reflects the HTTPS url of the project.

## Adding IdentityServer
IdentityServer is based on OWIN/Katana and distributed as a NuGet package. To add it to the newly created web host, install the following two packages:

````
Install-Package Microsoft.Owin.Host.Systemweb
Install-Package IdentityServer3
````

## Configuring IdentityServer - Clients
IdentityServer needs some information about the clients it is going to support, this can be simply supplied using a `Client` object:

```csharp
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
```

A special setting here is the `AllowedCorsOrigins` property. This allows IdentityServer to only accept requests from registered URLs. More on the `popup.html` later.

**Remark** Right now the client has access to all scopes (via the `AllowAccessToAllScopes` setting). For production applications you would lock that down.

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
Finally we will add scopes to IdentityServer. For authentication we'll only put standard IODC scopes. When we'll integrate API calls we'll create our own.

```csharp
public static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            StandardScopes.OpenId,
            StandardScopes.Profile,
            StandarsSc
        };
    }
}
```

## Adding Startup
IdentityServer is configured in the startup class. Here we provide information about the clients, users, scopes,
the signing certificate and some other configuration options.
In production you should load the signing certificate from the Windows certificate store or some other secured source.
In this sample we simply added it to the project as a file (you can download a test certificate from [here](https://github.com/identityserver/Thinktecture.IdentityServer3.Samples/tree/master/source/Certificates).
Add it to the project and set its build action to `Copy to output`.


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

    X509Certificate2 LoadCertificate()
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

 - [jQuery](http://jquery.com/download/)
 - [Bootstrap](http://getbootstrap.com/getting-started/#download)
 - [oidc-token-manager](https://github.com/IdentityModel/oidc-token-manager)

and put them in the `content` folder:

![content folder](https://cloud.githubusercontent.com/assets/6102639/12249132/e180dee4-b911-11e5-8f0b-41072202768c.png)

We also create a basic `index.html` file:

```html
<!DOCTYPE html>
<html>
<head>
    <title>JS Application</title>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="content/css/bootstrap.css" />
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

    <script src="content/js/jquery-2.1.4.js"></script>
    <script src="content/js/bootstrap.js"></script>
    <script src="content/js/oidc.js"></script>
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
    <script src="content/js/oidc.js"></script>
</body>
</html>
```

We have two HTML files because `oidc-token-manager` can open a popup to show the login form to the user.

## JS Client - authentication

Now that we have everything we need, we can configure our login settings in `index.html` thanks to the `OidcTokenManager` JS class.

```js
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

    filter_protocol_claims: true
};

var manager = new OidcTokenManager(settings);

$('.js-login').click(function () {
    manager.openPopupForTokenAsync()
        .then(function () {
            display('.js-id-token', manager.profile);
        }, function (error) {
            console.error(error);
        });
});
```

Let's go quickly through the settings:

 - `authority` is the base URL of our IdentityServer instance. This will allow oidc-token-manager to query the metadata endpoint so it can validate the tokens
 - `client_id` is the id of the client we want to use when hitting the authorization endpoint
 - `popup_redirect_uri` is the redirect URL used when using the `openPopupForTokenAsync` method. If you prefer not having a popup and redirecting the user in the main window, you can use the `redirect_uri` property and the `redirectForTokenAsync` method
 - `response_type` defines in our case that we only expect an identity token back
 - `scope` defines the scopes the application asks for
 - `filter_protocol_claims` instructs oidc-token-manager if it has to filter some OIDC protocol claims from the response: `nonce`, `at_hash`, `iat`, `nbf`, `exp`, `aud`, `iss` and `idp`

We also have to configure `popup.html`:

```js
var manager = new OidcTokenManager();
manager.processTokenPopup();
```

Under the hoods, the instance of `OidcTokenManager` of `index.html` redirects the popup to the login page. When IdentityServer redirects the user to the popup page, the information is then passed back to the main page.
Also note that the method used to open the popup and login returns promises, which is why we use `then`. A lot of methods of oidc-token-manager use this pattern.

At this stage, you can login:

![login-popup](https://cloud.githubusercontent.com/assets/6102639/12250630/94dce974-b91c-11e5-8077-dabcf88d80e5.png)

![login-complete](https://cloud.githubusercontent.com/assets/6102639/12250642/a703c0aa-b91c-11e5-9e3f-2b2b70b10e5b.png)

You can try and set the `filter_protocol_claims` property to `false` and see the additional claims being stored in the `profile` property.

## JS application - scopes

Remember we configured our user with an `email` claim? It doesn't show up in the identity token because the scopes the client asked for - `openid` and `profile` - don't contain this claim.
If you want to get the user's email, you'll have to ask for another scope which is called `email` by editing the `scopes` property of the `OidcTokenManager` settings.

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
            StandardScopes.Email
        };
    }
}
```

We don't have to change anything in the client configuration because we specified it has access to all the scopes. In a realistic scenario, you want to give a client access to only the scopes it's expected to request, so this would require a change in the client configuration as well.

After this, we can see the `email` claim in the identity token:

![login-email](https://cloud.githubusercontent.com/assets/6102639/12251318/25d3ce20-b922-11e5-9867-efc6afb76aea.png)

# Part 2 - API call
In the second part, we'll see how you can call a protected API from the JS application.
This will require us to obtain an access token from IdentityServer when we login and use it when calling the API.

## Create the API project
Create a new empty web application in Visual Studio, still selecting "No authentication".

![create api](https://cloud.githubusercontent.com/assets/6102639/12252754/251cd1f0-b92e-11e5-8f8f-469cfc2a0103.png)

## Configuring the API
For this example, we'll create a very simple API based on ASP.NET Web API.
To do so, install the following packages:

```
Install-Package Microsoft.Owin.Host.SystemWeb -ProjectName Api
Install-Package Microsoft.Owin.Cors -ProjectName Api
Install-Package Microsoft.AspNet.WebApi.Owin -ProjectName Api
Install-Package IdentityServer3.AccessTokenValidation -ProjectName Api

Update-Package -ProjectName Api
```

Let's now create a `Startup` class and build our OWIN pipeline.

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

Since the JS application will make the call to the API, CORS must be enabled on the API. In our case, we allow all origins to access it

We then use token validation provided by the `IdentityServer3.AccessTokenValidation` package. By setting the `Authority` property, the metadata document will be retrieved and used to configure the token validation settings.
Since version 2.2, IdentityServer implements the [introspection endpoint](../endpoints/introspection.html) to validate tokens. This endpoint requires scope authentication which makes it more secured than the traditional access token validation endpoint.

Finally, we add our Web API configuration. Note the use of a global `AuthorizeAttribute` which makes every endpoint of the API only accessible to authenticated requests.

Let's now add a basic endpoint to our API:

```csharp
[Route("values")]
public class ValuesController : ApiController
{
    public IEnumerable<string> Get()
    {
        var random = new Random();

        return new[]
        {
            random.Next(0, 10).ToString(),
            random.Next(0, 10).ToString()
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
As we did for the identity token, let's have another section where we'll see what the access token contains:

```html
[...]
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
        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">ID Token Contents</div>
                <div class="panel-body">
                    <pre class="js-id-token"></pre>
                </div>
            </div>
        </div>

        <div class="col-xs-6">
            <div class="panel panel-default">
                <div class="panel-heading">Access Token</div>
                <div class="panel-body">
                    <pre class="js-access-token"></pre>
                </div>
            </div>
        </div>
    </div>
</div>
[...]
<script>
    var settings = {
        authority: 'https://localhost:44300',
        client_id: 'js',
        popup_redirect_uri: 'http://localhost:56668/popup.html',

        response_type: 'id_token token',
        scope: 'openid profile email api',

        // this will allow all the OIDC protocol claims to be visible in the window. normally a client app
        // wouldn't care about them or want them taking up space
        filter_protocol_claims: true
    };

    var manager = new OidcTokenManager(settings);

    $('.js-login').click(function () {
        manager.openPopupForTokenAsync()
            .then(function () {
                display('.js-id-token', manager.profile);
                display('.js-access-token', manager.access_token && { access_token: manager.access_token, expires_in: manager.expires_in } || '');
            }, function (error) {
                console.error(error);
            });
    });
</script>
```

The modifications include:

 - a new panel to show the access token
 - an updated `response_type` to specify we want an access token back along with the identity token
 - the new `api` scope to be requested as part of the login request

After logging in, here's what the get:

![access token](https://cloud.githubusercontent.com/assets/6102639/12253628/1bc4e550-b935-11e5-95e9-fca21057cd08.png)

## Calling the API

We can modify the JS application to call the API:

```html
[...]
<div class="container main-container">
    <div class="row">
        <div class="col-xs-12">
            <ul class="list-inline list-unstyled requests">
                <li><a href="index.html" class="btn btn-primary">Home</a></li>
                <li><button type="button" class="btn btn-default js-login">Login</button></li>
                <li><button type="button" class="btn btn-default js-call-api">Call API</button></li>
            </ul>
        </div>
    </div>

    <div class="row">
        <div class="col-xs-4">
            <div class="panel panel-default">
                <div class="panel-heading">ID Token Contents</div>
                <div class="panel-body">
                    <pre class="js-id-token"></pre>
                </div>
            </div>
        </div>

        <div class="col-xs-4">
            <div class="panel panel-default">
                <div class="panel-heading">Access Token</div>
                <div class="panel-body">
                    <pre class="js-access-token"></pre>
                </div>
            </div>
        </div>

        <div class="col-xs-4">
            <div class="panel panel-default">
                <div class="panel-heading">API call result</div>
                <div class="panel-body">
                    <pre class="js-api-result"></pre>
                </div>
            </div>
        </div>
    </div>
</div>
[...]
$('.js-call-api').click(function () {
    var headers = {};
    if (manager.access_token) {
        headers['Authorization'] = 'Bearer ' + manager.access_token;
    }

    $.ajax({
        url: 'http://localhost:60136/values',
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

We now have a button which will trigger the API call along with another panel to show the call response.
It's worth noting that the access token is passed in the `Authorization` header of the request.

Here's what we see if we call the API prior to login:

![api without access token](https://cloud.githubusercontent.com/assets/6102639/12254749/48aab0c6-b940-11e5-8842-83a526a5e316.png)

And after login:

![api with access token](https://cloud.githubusercontent.com/assets/6102639/12254750/497df5f8-b940-11e5-9cca-7ee1184605e5.png)

In the first case, there was no token, so the access token validation middleware did nothing. The request flowed through the API as unauthenticated, and the global `AuthorizeAttribute` rejected it and responded with a `401 Unauthorized` error.
In the second case, the token validation middleware found the token in the `Authorization` header, passed it along to the introspection endpoint which flagged it as valid, and created an identity with the claims it contained. The request, this time authenticated, flowed to Web API, the `AuthorizeAttribute` contrainsts were staisfied, and the endpoint was invoked.

# Part 3 - Renewing tokens, checking session and logging out

We now have a working JS application which logs in against IdentityServer and make successful calls to a protected API.
Users will encounter issues when their access token expires and is rejected by the access token validation middleware on the API.
To work around that, we can setup oidc-token-manager to automatically renew the access token when it's about to expire, without any steps required for the user.

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
                AccessTokenLifetime = 70
            }
        };
    }
}
```

The access token lifetime, which is 1 hour by default, has been changed to 70 seconds.  
What you'll experience if you login the JS application again is that you'll get the same `401 Unauthorized` error when you call the API 10 seconds after logging in.

## Renewing tokens

We are going to rely on a feature oidc-token-manager gives us to renew the tokens. The idea is to have a hidden `iframe` on our page, which oidc-token-manager will use to issue a new authorization request while the user is still logged in. The [`prompt`](../endpoints/authorization.html) setting is used to prevent the user from having to log in or give his consent while he has a valid session.  
IdentityServer will return a new access token which will replace the one stored in the `OidcTokenManager` instance.

To achieve this, we have several steps to take.

First, we have to create a new file, `silent-renew.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
    <script src="content/js/oidc.js"></script>
    <script>
        var manager = new OidcTokenManager();
        manager.processTokenCallbackSilent();
    </script>
</body>
</html>
```

and modify the token manager settings:

```js
var settings = {
    authority: 'https://localhost:44300',
    client_id: 'js',
    popup_redirect_uri: 'http://localhost:56668/popup.html',

    silent_renew: true,
    silent_redirect_uri: 'http://localhost:56668/silent-renew.html',

    response_type: 'id_token token',
    scope: 'openid profile email api',

    // this will allow all the OIDC protocol claims to be visible in the window. normally a client app
    // wouldn't care about them or want them taking up space
    filter_protocol_claims: true
};
```

The HTML file will be used in a dynamically created `iframe` to renew the token. The URL of that page is passed to the token manager settings to instruct it to automatically renew the token when it expires.  
The token manager will renew the token 60 seconds before it expires, which explains why we chose to change to access token lifetime to 70 seconds.  
This means the token will be renewed every 10 seconds.

The second step is to let IdentityServer know that it is OK to redirect the user to it after the authentication is successful:

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

The third and final step will allow to see the updated access token in the associated panel every time it's renrewed.  
The `OidcTokenManager` exposes a `addOnTokenObtained` method which takes a callback as a parameter. This callback will be invoked every time an access token is obtained. This includes the first time th euser logs in as well as the next times when the token is silently renewed.

```js
var manager = new OidcTokenManager(settings);

manager.addOnTokenObtained(function () {
    display('.js-access-token', manager.access_token && { access_token: manager.access_token, expires_in: manager.expires_in } || '');
});

$('.js-login').click(function () {
    manager.openPopupForTokenAsync()
        .then(function () {
            display('.js-id-token', manager.profile);

            // Removed
            // display('.js-access-token', manager.access_token && { access_token: manager.access_token, expires_in: manager.expires_in } || '');
        }, function (error) {
            console.error(error);
        });
});
```

You can now see that the access token is renewed every 10 seconds.