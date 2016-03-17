---
layout: docs-default
---

# Custom Grant Types

The token endpoint allows for extensibility using custom grant types.

The following is an example for a token request using a custom grant type:

```
POST /connect/token
Authorization: Basic xxx:yyy

grant_type=my_custom_credential&
scope=api1&
my_credential=foobar&
some_other_parameter=quux&
```

Using IdentityServer's extensibility mechanism, you can register a custom grant validator for the `my_custom_credential`.
The job of a custom grant validator is to validate the incoming data, and map that to an IdentityServer user.

You start by implementing this interface:

```csharp
public interface ICustomGrantValidator
{
    Task<CustomGrantValidationResult> ValidateAsync(ValidatedTokenRequest request);
    string GrantType { get; }
}
```

In the `GrantType` property you specify which custom grant type you want to handle with this validator. 
In the ValidateAsync method you have access to the raw requests (e.g. for reading custom parameters like in the example
above) as well as validated data like scopes and client identity.

The result object allows you to set either a principal (with claims) that map to a user - or an error message.

You register the validator by setting it on the service factory:

```csharp
factory.CustomGrantValidators.Add( 
    new Registration<ICustomGrantValidator, MyCustomGrantValidator>());
```

To use this grant type, you need to create a client with the following configuration:

* The `Flow` must be set to `Custom`
* The `AllowedCustomGrantTypes` must include the custom grant type

One typical use case for custom grants is to translate between token types (e.g. SAML to JWT or Facebook to JWT) thus
bridging the gap between two identity management systems.

See [rfc7521 - Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants](https://tools.ietf.org/html/rfc7521) 
for more information on this use case.
