---
layout: docs-default
---

#Creating the simplest OAuth2 Authorization Server, Client and API

The intention of this walkthrough is to create the simplest possible IdentityServer installation acting as an OAuth2 authorization server.
This is supposed to get you started with some of the basic features and configuration options.
There are other more advanced walk-throughs in the docs that you could do afterwards. This tutorial includes:

* Creating a self-hosted IdentityServer

* Setting up clients for application to service communication both using an application account as well as on behalf of a user

* Registering an API

* Requesting access tokens

* Calling an API

* Validating an access token

##Setting up IdentityServer
First we gonna create a console host and set up IdentityServer.

Start by creating a standard console application and add IdentityServer via nuget:

```
install-package Thinktecture.IdentityServer.v3 -pre
```

###Registering the API
APIs are modeled as scopes - you need to register all APIs that you want to be able to request access tokens for. For that we create a class that returns a list of `Scope`:

```csharp
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

###Registering the Client
For now we want to register a single client. This client will be able to request a token for the `api1` scope.
For our first iteration, there will be no human involved and the client will simply request the token
on behalf of itself (think machine to machine communication). Later we will add a user to the picture.

For this client we configure the following things:

* Display name and id (unique name)

* The client secret (used to authenticate the client against the token endpoint)

* The flow (client credentials flow in this case)

* Usage of so called reference tokens. Reference tokens do not need a signing certificate.

```csharp
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

###Configuring IdentityServer
IdentityServer gets configured as OWIN middleware. This happens in the `Startup` class using the `UseIdentityServer` extension method.
The following snippets sets up a barebones server with our scopes and clients. We also set up an empty list of users - we gonna add users later).

```csharp
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

###Hosting IdentityServer
The very last step is to host IdentityServer. For this we add the Katana self-hosting package to our console application:

```
install-package Microsoft.Owin.SelfHost
```

Add the following code `Program.cs`:

```csharp
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

##Adding an API
In this part we will add a simple web API that is configured to require token from the IdentityServer we just set up.

###Creating the Web Host
Add a new `ASP.NET Web Application` to the solution and choose the `Empty` option (no framework references).

Add the necessary nuget packages:

```
install-package Microsoft.Owin.Host.SystemWeb
install-package Microsoft.AspNet.WebApi.Owin
install-package Thinktecture.IdentityServer.v3.AccessTokenValidation -pre
```

###Adding a Controller
Add this simple test controller:

```csharp
[Route("test")]
public class TestController : ApiController
{
    public IHttpActionResult Get()
    {
        return Json(new { message = "OK" });
    }
}
```

###Adding Startup
Add the following `Startup` class for both setting up web api and configuring trust with IdentityServer

```csharp
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

##Adding a Console Client
In the next part we will add a simple console client that will request an access token and use that to communicate with the api.

First add a new console project and install a nuget package for an OAuth2 client helper library:

```
install-package Thinktecture.IdentityModel.Client
```

The first code snippet requests the access token using the client credentials:

```csharp
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
static void CallApi(TokenResponse response)
{
    var client = new HttpClient();
    client.SetBearerToken(response.AccessToken);

    Console.WriteLine(client.GetStringAsync("http://localhost:14869/test").Result);
}
```

If you call both snippets, you should see `{ message: "OK" }` in your console.

Mission accomplished.



