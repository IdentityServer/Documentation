---
layout: docs-default
---

#User Service

**The sample for this topic can be found [here](https://github.com/thinktecture/Thinktecture.IdentityServer.v3.Samples/tree/master/source/CustomUserService)**

IdentityServer3 defines the `IUserService` interface to abstract the underlying identity management system being used for users. It provides semantics for users to authenticate with local accounts as well as external accounts. It also provides identity and claims to IdentityServer needed for tokens and the user info endpoint. Additionally, the user service can control the workflow a user will experience at login time. 

The `IUserService` interface defines these methods:

* `AuthenticateLocalAsync`
 * This method gets called for local authentication (whenever the user uses the username and password dialog).
* `AuthenticateExternalAsync`
 * This method gets called when the user uses an external identity provider to authenticate.
* `PreAuthenticateAsync`
 * This method gets called before the login page is shown. This allows you to determine if the user should be authenticated by some out of band mechanism (e.g. client certificates or trusted headers).
* `GetProfileDataAsync`
 * This method is called whenever claims about the user are requested (e.g. during token creation or via the userinfo endpoint).
* `IsActiveAsync`
 * This method gets called whenever identity server needs to determine if the user is still considered valid or active (e.g. if the user's account has been deactivated since they logged in).
* `SignOutAsync`
 * This method gets called when the user signs out.


## Authentication

Three of the APIs on the `IUserService` model authentication: `AuthenticateLocalAsync`, `AuthenticateExternalAsync`, and `PreAuthenticateAsync`. All three of these APIs are passed a `SignInMessage` object which represents contextual information about the request to login. These contextual properties are typically passed from the client to the  [authorization endpoint](../endpoints/authorization.html).

The `SignInMessage` contains these properties which might be of interest to the user service.

* `ClientId`: The identifier for the client requesting the login.
* `IdP`: The external identity provider requested.
* `Tenant`: The tenant the user is expected to come from.
* `LoginHint`: The expected username the user will use to login.
* `DisplayMode`: The display mode passed from the authorization request.
* `UiLocales`: The UI locales passed from the authorization request.
* `AcrValues`: The acr values passed from the authorization request.

In addition to the `SignInMessage`, some of the authentication methods accept additional parameters.

`AuthenticateLocalAsync` is invoked if the user enteres credentials into the local login page in IdentityServer. The username and password are parameters, as well as the `SignInMessage`.

`AuthenticateExternalAsync` is invoked when an external [identity provider](../configuration/identityProviders.html) is used to authenticate. The `ExternalIdentity` is passed an `ExternalIdentity`, as well as the `SignInMessage`. 

The `ExternalIdentity` contains:

* `Provider`: Identifier of the external identity provider.
* `ProviderId`: User's unique identifier provided by the external identity provider.
* `Claims`: Claims supplied for the user from the external identity provider.

`PreAuthenticateAsync` is invoked prior to showing the login page and allows a custom user service to prevent the login page from being displayed. This is typically due to some outside parameter that informs the user service that the user should already be logged in. It is only passed the `SignInMessage`.

### Authentication outcomes

The return value of all of the authentication methods is an `AuthenticateResult`. The returned `AuthenticateResult` indicates one of many possible outcomes. The constructor is overloaded and the one used indicates which of these outcomes is chosen. The outcomes are:

#### Full login

To fully log the user in the authentication API must produce a `subject` and a `name` that represent the user. The `subject` is the user service's unique identifier for the user and the `name` is a display name for the 
user that will be displayed in the user interface.

Optionally a list of `Claim` can also be provided. These claims can be any additional values that might be needed by the user service inthe other APIs on the user service.

If the user is being logged in and they used an external identity provider, then the `identityProvider` parameter should also be indicated. This will be included as the `idp` claim in the identity token and the user info endpoint. If the user is being authenticated with a local account, then this parameter should not be used (and let the default of `Constants.BuiltInIdentityProvider` be used instead).

There is also an optional `authenticationMethod` parameter which populates the `amr` claim. This can be used to indicate how the user authenticated, such as two factor authentication, or client certificates. If it is not passed, then `password` is assumed for local logins. For external logins, then the value of `external` will automatically be used to indicate an external authentication. 

The entirety of these claims (`subject`, `name`, `idp`, `amr` and the list of `Claim`) are used to populate the authentication cookie that is issued for the user for IdentityServer. This authentication cookie is issued and managed by using the Katana cookie authentication middleware with an `AuthenticationType` indicated by the constant `Constants.PrimaryAuthenticationType`.

The `ClaimsPrincipal` that is created from the full login is then used as the parameter for the other APIs on the `IUserService`. These APIs are `GetProfileDataAsync`, `IsActiveAsync`, and `SignOutAsync`.

#### Partial login (with redirect)

In addition to a full login, the authentication APIs can perform a "partial login". A partial login indicates that the user has proven their identity, has a local account, but is not yet allowed to continue. They must first perform some other action or provide some other data before being allowed to login fully. This is useful to customize the user's workflow before allowing them to fully login. This could be useful to force a user to fill in a registration page, change a password, or accept a EULA before letting them continue.

This partial login is performed by issuing a "partial login" cookie using the Katana cookie authentication middleware with an `AuthenticationType` indicated by the constant `Constants.PartialSignInAuthenticationType`.

The partial login is indicated with all of the same parameters as described above for a full login (i.e. `subject`, `name`, `claims`, `amr`, and `idp`) as well as a `redirectPath`. The `redirectPath` represents a custom web page provided by the hosting application that the user will be redirected to. On that web page the user's `subject` and `name` claims can be used to identify the user (in addition to all the other claims indicated). In order to obtain these claims, the page must use the Katana authentication middleware to authenticate against the `Constants.PartialSignInAuthenticationType` authentication type, or simply be executing within the same cookie path as IdentityServer on the web server.

Once the user has completed their work on the custom web page, they can be redirected back to IdentityServer to continue with the full login process. The URL to redirect the user back to provided as a claim in the `Constants.PartialSignInAuthenticationType` authentication type and is identified by the claim type `Constants.ClaimTypes.PartialLoginReturnUrl`.

#### External login (with redirect)

It's possible that the user has performed an external authentication, but there is no local account for the user. A custom user service can choose to redirect the user without a local account. This is performed by creating a `AuthenticateResult` with a redirect and the `ExternalIdentity` passed to the `AuthenticateExternalAsync` API. This performs a partial login (as above via the `PartialSignInAuthenticationType`) but there is no subject claim in the issued cookie. Instead there is a claim of type `external_provider_user_id` (or via `Constants.ClaimTypes.ExternalProviderUserId`) whose `Issuer` is the external provider identifier and whose value is the external provider's identifier for the user. These values can then be used to create a local account and associate the external account.

Once the user has completed their registration on the custom web page, they can be redirected back to IdentityServer via the same `PartialLoginReturnUrl` as described above.

#### Login error

Finally, the authentication APIs can provide an error that will be displayed on the login view. This is indicated by creating the `AuthenticateResult` using the constructor that accepts a `string` (the error) as its argument.
