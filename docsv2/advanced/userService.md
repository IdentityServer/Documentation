---
layout: docs-default
---

# User Service

**The sample for this topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/CustomUserService)**

IdentityServer3 defines the `IUserService` interface to abstract the underlying identity management system being used for users. It provides semantics for users to authenticate with local accounts as well as external accounts. It also provides identity and claims to IdentityServer needed for tokens and the user info endpoint. Additionally, the user service can control the workflow a user will experience at login time before being allowed to return to the client application (e.g. for things such as additional login requirements such as 2fa or other custom requirements such as accepting a EULA).

The methods on the user service are broken down into methods that relate to authentication and methods that relate to the user's profile and issuing claims for tokens.

* The authentication methods are called by IdentityServer when the user is attempting to authenticate. The outcome of these methods can either be a successful login, an error, or a partial login (see below for more information on partial logins).
* The profile related methods are invoked when IdentityServer needs claims or needs to ensure the user is still allowed to receive tokens.

## IUserService
The `IUserService` interface defines these methods:

* `PreAuthenticateAsync`
    * This method is called before the login page is shown. This allows the user service to determine if the user is already authenticated by some out of band mechanism (e.g. client certificates or trusted headers) and prevent the login page from being shown.
    * Passed a `PreAuthenticationContext` with these properties:
        * `SignInMessage`: The contextual information passed to the authorize endpoint.
        * `AuthenticateResult`: The user service should assign to indicate the authentication outcome (or `null` to indicate the normal login page should be displayed).
        * `ShowLoginPageOnErrorResult` (added in v2.4): Set if the `AuthenticateResult` represents an error and you wish that the error is displayed on the login page (as opposed to the general error page).
* `AuthenticateLocalAsync`
    * This method is called for local authentication (whenever the user uses the username and password dialog). 
    * Passed a `LocalAuthenticationContext` with these properties:
        * `Username`: The username.
        * `Password`: The password.
        * `SignInMessage`: The contextual information passed to the authorize endpoint.
        * `AuthenticateResult`: The user service should assign to indicate the authentication outcome (or `null` to indicate invalid `Username` or `Password`).
* `AuthenticateExternalAsync`
    * This method is called when the user uses an external identity provider to authenticate. Allows the user service to map an external user to a local user. 
    * Passed a `ExternalAuthenticationContext` with these properties:
        * `ExternalIdentity`: Information from the external provider about the user. It contains:
            * `Provider`: Identifier of the external identity provider.
            * `ProviderId`: User's unique identifier provided by the external identity provider.
            * `Claims`: Claims supplied for the user from the external identity provider.
        * `SignInMessage`: The contextual information passed to the authorize endpoint.
        * `AuthenticateResult`: The user service should assign to indicate the authentication outcome (or `null` to indicate an error that there is no user matching the information provided in the `ExternalIdentity`).
* `PostAuthenticateAsync`
    * This method is called after the user has successfully authenticated but before they are returned to the client application. It allows a consolidated place to check for custom user workflows after all of the other authentication methods.
    * Passed a `PostAuthenticationContext` with these properties:
        * `SignInMessage`: The contextual information passed to the authorize endpoint.
        * `AuthenticateResult`: The current `AuthenticateResult`. The user service can re-assign to a non-`null` value to change the authentication outcome.
* `SignOutAsync`
    * This method is called when the user signs out.
    * Passed a `SignOutContext` with these properties:
        * `Subject`: The user that is signing out.
* `GetProfileDataAsync`
    * This method is called whenever claims about the user are requested (e.g. during token creation or via the userinfo endpoint).
    * Passed a `ProfileDataRequestContext` with these properties:
        * `Subject`: The user for which the profile is being requested.
        * `IssuedClaims`: The user service should assign to indicate the claims to be issued.
        * `RequestedClaimTypes`: The list of claim types requested. The user service should only return claim types as indicated by this property.
        * `AllClaimsRequested`: All claims should be returned (thus ignoring `RequestedClaimTypes`).
        * `Client`: The client making the request.
        * `Caller`: The caller within IdentityServer that is making the call. Value will be one of:
            * `"ClaimsProviderIdentityToken"`
            * `"ClaimsProviderAccessToken"`
            * `"UserInfoEndpoint"`
* `IsActiveAsync`
    * This method is called whenever identity server needs to determine if the user is still considered valid or active (e.g. if the user's account has been deactivated since they logged in).
    * Passed a `IsActiveContext` with these properties:
        * `Subject`: The user that is signing out.
        * `Client`: The client making the request.
        * `IsActive`: The user service should set to `false` if the user is no longer allowed to receive tokens.

### SignInMessage
The authentication methods all receive a `SignInMessage` in their context which contains these properties:

* `ReturnUrl`: The URL from which the user is making the login request.
* `ClientId`: The identifier for the client requesting the login.
* `IdP`: The external identity provider requested.
* `Tenant`: The tenant the user is expected to come from.
* `LoginHint`: The expected username the user will use to login.
* `DisplayMode`: The display mode passed from the authorization request.
* `UiLocales`: The UI locales passed from the authorization request.
* `AcrValues`: The acr values passed from the authorization request.


### AuthenticateResult

The return value of all of the authentication methods is an `AuthenticateResult`. The returned `AuthenticateResult` indicates one of many possible outcomes. The constructor is overloaded and the one used indicates which of these outcomes is chosen. The list is:

```csharp
// Full login
AuthenticateResult(string subject, string name, IEnumerable<Claim> claims = null, string identityProvider = Constants.BuiltInIdentityProvider, string authenticationMethod = null)

// Partial Login (where subject is known)
AuthenticateResult(string redirectPath, string subject, string name, IEnumerable<Claim> claims = null, string identityProvider = Constants.BuiltInIdentityProvider, string authenticationMethod = null)

// Partial Login (where subject is not known)
AuthenticateResult(string redirectPath, IEnumerable<Claim> claims)

// Partial Login (from external login)
AuthenticateResult(string redirectPath, ExternalIdentity externalId)

// Login error
AuthenticateResult(string errorMessage)
```

#### Full login

To fully log the user in the authentication API must produce a `subject` and a `name` that represent the user. The `subject` is the user service's unique identifier for the user and the `name` is a display name for the 
user that will be displayed in the user interface.

Optionally a list of `Claim` can also be provided. These claims can be any additional values that might be needed by the user service in the other APIs on the user service.

If the user is being logged in and an external identity provider was used, then the `identityProvider` parameter should also be specified. This will be included as the `idp` claim in the identity token and the user info endpoint. If the user is being authenticated with a local account, then this parameter should not be used (and let the default of `Constants.BuiltInIdentityProvider` be used instead).

There is also an optional `authenticationMethod` parameter which populates the `amr` claim. This can be used to indicate how the user authenticated, such as two factor authentication, or client certificates. If it is not passed, then `password` is assumed for local logins. For external logins, then the value of `external` will automatically be used to indicate an external authentication. 

The entirety of these claims (`subject`, `name`, `idp`, `amr` and the optional list of `Claim`) are used to populate the authentication cookie that is issued for the user for IdentityServer. This authentication cookie is issued and managed by using the Katana cookie authentication middleware with an `AuthenticationType` indicated by the constant `Constants.PrimaryAuthenticationType`.

The `ClaimsPrincipal` that is created from the full login is then used as the `Subject` for the other APIs on the `IUserService`. These APIs are `PostAuthenticateAsync`, `GetProfileDataAsync`, `IsActiveAsync`, and `SignOutAsync`.

#### Partial login

In addition to a full login, the authentication APIs can perform a "partial login". A partial login allows the user service to interrupt the user's login workflow and redirect them to a custom page where they must perform some action before they can continue to login (e.g. performing 2fa, completing a registration form, or accepting a EULA).

This partial login is performed by issuing a "partial login" cookie using the Katana cookie authentication middleware with an `AuthenticationType` indicated by the constant `Constants.PartialSignInAuthenticationType`.

The partial login can be created with all of the same parameters as described above for a full login (i.e. `subject`, `name`, `claims`, `amr`, and `idp`) as well as a `redirectPath`. Alternatively, the partial login can be created with just a `claims` collection as well as a `redirectPath`. The main difference is if the user's identity (subject) has been determined.

The `redirectPath` represents a custom web page provided by the hosting application that the user will be redirected to. On that web page the user's claims can be used to complete the custom workflow. In order to obtain these claims, the page can use the `GetIdentityServerPartialLoginAsync` [OWIN environment extension method](owin.html).

Once the user has completed their work on the custom web page, they can be redirected back to IdentityServer to continue with the full login process. The URL to redirect the user back to can be obtained via the `GetPartialLoginResumeUrlAsync` [OWIN environment extension method](owin.html). If the page needs to send the user back to the login page, then the `GetPartialLoginRestartUrlAsync` [OWIN environment extension method](owin.html) can be used instead.

Before redirecting the user back into IdentityServer, the claims in the partial login can be changed with the `UpdatePartialLoginClaimsAsync` [OWIN environment extension method](owin.html). If the page needs to remove or clear the partial login, then the `RemovePartialLoginCookie` [OWIN environment extension method](owin.html) can be used.

#### Partial Login (from external login)

It's possible that the user has performed an external authentication, but there is no local account for the user. A custom user service can choose to redirect the user without a local account. This is performed by creating a `AuthenticateResult` with a redirect and the `ExternalIdentity` passed to the `AuthenticateExternalAsync` API. This performs a partial login (as above via the `PartialSignInAuthenticationType`) but there is no subject claim in the issued cookie. Instead there is a claim of type `external_provider_user_id` (or via `Constants.ClaimTypes.ExternalProviderUserId`) whose `Issuer` is the external provider identifier and whose value is the external provider's identifier for the user. These values can then be used to create a local account and associate the external account.

Once the user has completed their registration on the custom web page, they can be redirected back to IdentityServer via the same mechanisms as described above.

#### Login error

Finally, the authentication APIs can provide an error that will be displayed on the login view. This is indicated by creating the `AuthenticateResult` using the constructor that accepts a `string` (the error) as its argument.
