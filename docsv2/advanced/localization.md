---
layout: docs-default
---

# Localization of messages

Messages that IdentityServer creates can be localized (or simply replaced) by implementing the `ILocalizationService` interface. It defines a single API:

* `string GetString(string category, string id)`

This API is passed a `category` for the message. IdentityServer only defines three categories:

```csharp
public static class LocalizationCategories
{
    public const string Messages = "Messages";
    public const string Events = "Events";
    public const string Scopes = "Scopes";
}
```

The `id` parameter then indicates the specific message from that category. The various identifiers that would be used are defined by constants in the `IdentityServer3.Core.Resources` namespace. For example, here is a short snippet of those constants (consult the code for the full list):

```csharp
namespace IdentityServer3.Core.Resources
{
	public class MessageIds
	{
			public const string ClientIdRequired = "ClientIdRequired";
			public const string ExternalProviderError = "ExternalProviderError";
			public const string Invalid_scope = "Invalid_scope";
			public const string InvalidUsernameOrPassword = "InvalidUsernameOrPassword";
			public const string MissingClientId = "MissingClientId";
			// ...
  }
}
```

The default implementation of the `ILocalizationService` loads all of its messages from embedded resource files (".resx") contained within IdentityServer itself. These embedded resource files only contain English strings.
