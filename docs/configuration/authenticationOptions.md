---
layout: docs-default
---

#Authentication Options

**The sample for this topic can be found [here](https://github.com/IdentityServer/IdentityServer3.Samples/tree/master/source/CustomUserService)**

The `AuthenticationOptions` is a property on the `IdentityServerOptions` to customize the login and logout views and behavior.

* `EnableLocalLogin`
   * Indicates if IdentityServer will allow users to authenticate with a local account. Disabling this setting will not display the username/password form on the login page. This also will disable the resource owner password flow. Defaults to `true`.
* `EnableLoginHint`
   * Indicates whether the `login_hint` parameter is used to prepopulate the username field. Defaults to `true`.
* `LoginPageLinks`
   * List of `LoginPageLink` objects. These allow the login view to provide the user custom links to other web pages that they might need to visit before they can login (such as a registration page, or a password reset page). 
   * `LoginPageLink` contains:
      * `Text`: The text to appear in the link.
      * `Href`: The URL for the `href` of the link.
   * The custom web page represented by the `LoginPageLink` would be provided by the hosting application. Once it has performed its task then it can resume the login workflow by redirecting the user back to the login view.
   * When a user follows one of the `LoginPageLink`s, a `signin` query string parameter is passed to the page. This parameter should be echoed back as a `signin` query string parameter to the login page when the user wishes to resume their login. The login view is located at the path "~/login" relative to IdentityServer's application base. 
* `RememberLastUsername`
   * Indicates whether IdentityServer will remember the last username entered on the login page. Defaults to `false`.
* `IdentityProviders`
   * Allows configuring additional identity providers - see [here](identityProviders.html).
* `CookieOptions`
    * `CookieOptions` object that configures how cookies are managed by IdentityServer. 
    * `CookieOptions` has these properties:
       * `Prefix`: Allows setting a prefix on cookies to avoid potential conflicts with other cookies with the same names. By default no prefix is used.
       * `ExpireTimeSpan`: The expiration duration of the authentication cookie. Defaults to `10` hours.
       * `IsPersistent`: Indicates whether the authentication cookie is marked as persistent. Defaults to `false`.
       * `SlidingExpiration`: Indicates if the authentication cookie is sliding, which means it auto renews as the user is active. Defaults to `false`.
       * `Path`: Sets the cookie path. Defaults to the base path of IdentityServer in the hosting application.
       * `AllowRememberMe`: Indicates whether the "remember me" option is presented to users on the login page. If selected this option will issue a persistent authentication cookie. Defaults to `true`.
          * If this setting is in use then the user's decision (either yes or no) will override the `IsPersistent` setting. In other words, if both `IsPersistent` and `AllowRememberMe` is enabled and the user decides to not remember their login, then no persistent cookie will be issued.
       * `RememberMeDuration`: Duration of the persistent cookie issued by the "remember me" option on the login page. Defaults to `30` days.
       * `SecureMode`: Gets or sets the mode for issuing the secure flag on the cookies issued. Defaults to SameAsRequest.
* `EnableSignOutPrompt`
   * Indicates whether IdentityServer will show a confirmation page for sign-out. When a client initiates a sign-out, by default IdentityServer will ask the user for confirmation. This is a mitigation technique against "logout spam". Defaults to `true`.
* `EnablePostSignOutAutoRedirect`
  * Gets or sets a value indicating whether IdentityServer automatically redirects back to a validated `post_logout_redirect_uri` passed to the signout endpoint. Defaults to `false`.
* `PostSignOutAutoRedirectDelay`
  * Gets or sets the delay (in seconds) before redirecting to a `post_logout_redirect_uri`. Defaults to `0`.
* `SignInMessageThreshold`
  * Gets or sets the limit after which old signin messages (cookies) are purged. Defaults to `5`.
* `InvalidSignInRedirectUrl`
  * Gets or sets the invalid sign in redirect URL. If the user arrives at the login page without a valid sign-in request, then they will be redirected to this URL. The URL must be absolute or can relative URLs (starting with "~/").
