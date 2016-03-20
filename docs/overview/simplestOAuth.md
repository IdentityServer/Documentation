---
layout: docs-default
---

# Creating the simplest OAuth2 Authorization Server, Client and API

The intention of this walkthrough is to create the simplest possible IdentityServer installation acting as an OAuth2 authorization server.
This is supposed to get you started with some of the basic features and configuration options (the full source code can be found [here](https://github.com/IdentityServer/Thinktecture.IdentityServer3.Samples/tree/master/source/Simplest%20OAuth2%20Walkthrough)).
There are other more advanced walk-throughs in the docs that you could do afterwards. This tutorial includes:

* Creating a self-hosted IdentityServer

* Setting up clients for application to service communication both using an application account as well as on behalf of a user

* Registering an API

* Requesting access tokens

* Calling an API

* Validating an access token

## Setting up IdentityServer
First we will create a console host and set up IdentityServer.

Start by creating a standard console application and add IdentityServer via nuget:

```
install-package Thinktecture.IdentityServer3
```

### Registering the API
APIs are modeled as scopes - you need to register all APIs that you want to be able to request access tokens for. For that we create a class that returns a list of `Scope`:

```csharp
using Thinktecture.IdentityServer.Core.Models;

static class Scopes
{
    public static List<Scope> Get()
    {
        return new List<Scope>
        {
            new Scope
            {
                Name = "api1"
            }
        };
    }
}
```

### Registering the Client
For now we want to register a single client. This client will be able to request a token for the `api1` scope.
For our first iteration, there will be no human involved and the client will simply request the token
on behalf of itself (think machine to machine communication). Later we will add a user to the picture.

For this client we configure the following things:

* Display name and id (unique name)

* The client secret (used to authenticate the client against the token endpoint)

* The flow (client credentials flow in this case)

* Usage of so called reference tokens. Reference tokens do not need a signing certificate.

```csharp
using Thinktecture.IdentityServer.Core.Models;

static class Clients
{
    public static List<Client> Get()
    {
        return new List<Client>
        {
            // no human involved
            new Client
            {
                ClientName = "Silicon-only Client",
                ClientId = "silicon",
                Enabled = true,
                AccessTokenType = AccessTokenType.Reference,

                Flow = Flows.ClientCredentials,
                ClientSecrets = new List<ClientSecret>
                {
                    new ClientSecret("F621F470-9731-4A25-80EF-67A6F7C5F4B8".Sha256())
                }
            }
        };
    }
}
```

### Configuring IdentityServer
IdentityServer is implemented as OWIN middleware. It is configured in  in the `Startup` class using the `UseIdentityServer` extension method.
The following snippets sets up a bare bones server with our scopes and clients. We also set up an empty list of users - we will add users later.

```csharp
using Thinktecture.IdentityServer.Core.Configuration;
using Thinktecture.IdentityServer.Core.Services.InMemory;

class Startup
{
    public void Configuration(IAppBuilder app)
    {
        var factory = InMemoryFactory.Create(
            scopes:  Scopes.Get(),
            clients: Clients.Get(),
            users:   new List<InMemoryUser>());

        var options = new IdentityServerOptions
        {
            Factory = factory
        };

        app.UseIdentityServer(options);
    }
}
```

### Hosting IdentityServer
The very last step is to host IdentityServer. For this we add the Katana self-hosting package to our console application:

```
install-package Microsoft.Owin.SelfHost
```

Add the following code `Program.cs`:

```csharp
using Microsoft.Owin.Hosting;
using Thinktecture.IdentityServer.Core.Logging;

static void Main(string[] args)
{
    LogProvider.SetCurrentLogProvider(new DiagnosticsTraceLogProvider());

    using (WebApp.Start<Startup>("https://localhost:44333"))
    {
        Console.WriteLine("server running...");
        Console.ReadLine();
    }
}
```

When you run the console app, you should see some diagnostics output and `server running...`.

## Adding an API
In this part we will add a simple web API that is configured to require an access token from the IdentityServer we just set up.

### Creating the Web Host
Add a new `ASP.NET Web Application` to the solution and choose the `Empty` option (no framework references).

Add the necessary nuget packages:

```
install-package Microsoft.Owin.Host.SystemWeb
install-package Microsoft.AspNet.WebApi.Owin
install-package Thinktecture.IdentityServer3.AccessTokenValidation
```

### Adding a Controller
Add this simple test controller:

```csharp
[Route("test")]
public class TestController : ApiController
{
    public IHttpActionResult Get()
    {
        var caller = User as ClaimsPrincipal;

        return Json(new
        {
            message = "OK computer",
            client =  caller.FindFirst("client_id").Value
        });
    }
}
```

The `User` property on the controller gives you access to the claims from the access token.

### Adding Startup
Add the following `Startup` class for both setting up web api and configuring trust with IdentityServer

```csharp
using Thinktecture.IdentityServer.AccessTokenValidation;

public void Configuration(IAppBuilder app)
{
    // accept access tokens from identityserver and require a scope of 'api1'
    app.UseIdentityServerBearerTokenAuthentication(new IdentityServerBearerTokenAuthenticationOptions
        {
            Authority = "https://localhost:44333",
            RequiredScopes = new[] { "api1" }
        });

    // configure web api
    var config = new HttpConfiguration();
    config.MapHttpAttributeRoutes();

    // require authentication for all controllers
    config.Filters.Add(new AuthorizeAttribute());

    app.UseWebApi(config);
}
```

Try opening the browser and access the test controller - you should see a 401 because the necessary access token is missing.

## Adding a Console Client
In the next part we will add a simple console client that will request an access token and use that to authenticate with the api.

First add a new console project and install a nuget package for an OAuth2 client helper library:

```
install-package Thinktecture.IdentityModel.Client
```

The first code snippet requests the access token using the client credentials:

```csharp
using Thinktecture.IdentityModel.Client;

static TokenResponse GetToken()
{
    var client = new OAuth2Client(
        new Uri("https://localhost:44333/connect/token"),
        "silicon",
        "F621F470-9731-4A25-80EF-67A6F7C5F4B8");

    return client.RequestClientCredentialsAsync("api1").Result;
}
```

The second code snippet calls the API using the access token:

```csharp
using Thinktecture.IdentityModel.Client;

static void CallApi(TokenResponse response)
{
    var client = new HttpClient();
    client.SetBearerToken(response.AccessToken);

    Console.WriteLine(client.GetStringAsync("http://localhost:14869/test").Result);
}
```

If you call both snippets, you should see `{"message":"OK computer","client":"silicon"}` in your console.

## Adding a User
So far, the client requests an access token for itself and no user is involved. Let's introduce a human.

### Adding a user service
The user service manages users - for this sample we will use the simple in-memory user service.
First we need to define some users:

```csharp
using Thinktecture.IdentityServer.Core.Services.InMemory;
 
static class Users
{
    public static List<InMemoryUser> Get()
    {
        return new List<InMemoryUser>
        {
            new InMemoryUser
            {
                Username = "bob",
                Password = "secret",
                Subject = "1"
            },
            new InMemoryUser
            {
                Username = "alice",
                Password = "secret",
                Subject = "2"
            }
        };
    }
}
```

`Username` and `Password` are used to authenticate the user,
the `Subject` is the unique identifier for that user that will be embedded into the access token.

In `Startup` Configuration method replace the empty user list with a call the `Get` method.

Replace:
 ```csharp
            users: new List<InMemoryUser>());
 ```
 With:
  ```csharp
            users: Users.Get());
 ```
 
### Adding a Client
Next we will add a client definition that uses the flow called `resource owner password credential grant`.
This flow allows a client to send the user's username and password to the token service and get an access token back in return.

In total the `Clients` class looks like this then:

```csharp
using Thinktecture.IdentityServer.Core.Models;

static class Clients
{
    public static List<Client> Get()
    {
        return new List<Client>
        {
            // no human involved
            new Client
            {
                ClientName = "Silicon-only Client",
                ClientId = "silicon",
                Enabled = true,
                AccessTokenType = AccessTokenType.Reference,

                Flow = Flows.ClientCredentials,
                ClientSecrets = new List<ClientSecret>
                {
                    new ClientSecret("F621F470-9731-4A25-80EF-67A6F7C5F4B8".Sha256())
                }
            },

            // human is involved
            new Client
            {
                ClientName = "Silicon on behalf of Carbon Client",
                ClientId = "carbon",
                Enabled = true,
                AccessTokenType = AccessTokenType.Reference,

                Flow = Flows.ResourceOwner,
                ClientSecrets = new List<ClientSecret>
                {
                    new ClientSecret("21B5F798-BE55-42BC-8AA8-0025B903DC3B".Sha256())
                }
            }
        };
    }
}
```

### Updating the API
When a human is involved, the access token will contain the `sub` claim to uniquely identify the user.
Let's make this small modification to the API controller:

```csharp
[Route("test")]
public class TestController : ApiController
{
    public IHttpActionResult Get()
    {
        var caller = User as ClaimsPrincipal;

        var subjectClaim = caller.FindFirst("sub");
        if (subjectClaim != null)
        {
            return Json(new
            {
                message = "OK user",
                client = caller.FindFirst("client_id").Value,
                subject = subjectClaim.Value
            });
        }
        else
        {
            return Json(new
            {
                message = "OK computer",
                client = caller.FindFirst("client_id").Value
            });
        }
    }
}
```

### Updating the Client
Next add a new method to the client to request an access token on behalf of a user:

```csharp
static TokenResponse GetUserToken()
{
    var client = new OAuth2Client(
        new Uri("https://localhost:44333/connect/token"),
        "carbon",
        "21B5F798-BE55-42BC-8AA8-0025B903DC3B");

    return client.RequestResourceOwnerPasswordAsync("bob", "secret", "api1").Result;
}
```

Now try both methods of requesting a token and inspect the claims and the API response.

## What to do next
This walk-through covered a very simple OAuth2 scenario. Next you could try:

* The other flows - e.g. implicit, code or hybrid. They are all enablers for advanced scenarios like federation and external identities

* Connect to your user database - either by writing your own user service or by using our out of the box support for ASP.NET Identity and MembershipReboot

* Store client and scope configuration in a data store. We have out of the box support for Entity Framework.

* Add authentication and identity tokens using OpenID Connect and identity scopes
